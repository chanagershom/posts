![](/images/hybrid-container-vnf/containerized-hybrid-vnf.png)
## Hybrid VNF Container Orchestration 

The need to orchestrate virtual network functions that run in Linux hosted containers is an emerging challenge in the NFV world.  Driven by the need for high responsiveness and deployment density, as well as a desire to adopt modern microservices architectures, users and vendors are finding containers an appealing prospect.  In this post I address a hybrid VNF orchestration consisting of Kubernetes and Swarm container managers, as well as conventional hosts.  The idea is to explore the challenges and opportunities presented by such a scenario.  The world of containerized VNFs is somewhat limited (to say the least) that will run out of the box in Kubernetes and/or Docker Swarm, so I did this exploration with containerized versions of standard Linux network components, the Quagga router and Nginx configured as a load balancer.

### Target Architecture

![Architecture]()

The architecture is composed of some familiar building blocks; the [Kubernetes blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint), the [Kubernetes plugin](https://github.com/cloudify-examples/cloudify-kubernetes-plugin), the [Docker Swarm Blueprint](https://github.com/cloudify-examples/docker-swarm-blueprint), and the [Deployment Proxy plugin](https://github.com/cloudify-examples/cloudify-proxy-plugin).  In addition, a yet to be published Docker Swarm plugin is used.

The basic idea is to run network traffic through one VNF on Docker Swarm, then through another VNF in Kubernetes.  In this example, traffic is load balanced though an Nginx container on Swarm, which load balances a couple of VMs on the other side of a containerized Quagga instance.

![Packet Flow]()

### Automation Process

The first step in any automated orchestration is to identify (and verify when needed) the steps to create the desired end state.  The end state for Cloudify is defined by a TOSCA blueprint or blueprints.  Clearly a Kubernetes and Swarm cluster are needed, along with the requisite networking setup.  To reflect a reasonable production setup, these clusters should have separate deployment lifecycles, and therefore be modeled as separate blueprints.  The existing Kubernetes and Swarm cluster blueprints are a good starting point, but need some tweaking.  Then a third blueprint is needed to actually deploy and configure the containers on each of the clusters.

#### Kubernetes Cluster Blueprint

Whenever considering an orchestration plan that consists of multiple blueprints, a key factor to consider is the outputs.  The Cloudify deployment proxy achieves its aims by copying outputs from configured blueprints, so the containing blueprint can perform the tasks it needs.  In this case, the "containing blueprint" will be the blueprint that deploys the services onto the Kubernetes cluster and (potentially) gets the IP addresses of the target VMs running Apache.  The Kubernetes cluster URL is already in the outputs of the standard Kubernetes blueprint.  For convenience, the Kubernetes blueprint will be changed to create the `10.100.102.0/24` network and the Apache VMs.  By virtue of starting the VMs, the blueprint will have access to the IPs and of course the subnet (by virtue of a Cloud API, a host IP pool, or hard coding).  These can be then exposed in the outputs like so:

```yaml
outputs:
  apache_info:
    value:
      network: {get_property: [ apache_subnet, subnet, cidr ]
      ips: {concat: [ {get_attribute: [ apache_host1, ip ] }, "," , {get_attribute: [ apache_host2, ip ] } ] }
```

In addition to simply starting the instance, the Apache web server is started and supplied with an identifyable `index.html` so that load balancing can be verified via `curl`.  Also, to match the architecture, the Kubernetes blueprint must be changed to run the cluster in the `10.100.101.0/24` network.  To simplify the setup, the cluster will only have a single node that will contain the Quagga router, and have an interface to the `10.100.102.0/24` network.

### VNF Preparation

Quagga operates by manipulating the Linux kernel routing tables.  Unprivileged containers run in their own network namespace, and so won't affect the default tables.  To allow Quagga to access the routing tables, it must run in privileged mode, which is enabled by running the Kubernetes daemon (kubelet) with the `--allow-privileged` option.  The example Kubernetes blueprint already does this.  In addition to privileged mode is providing access to the host network stack.  This is covered in the section about the service blueprint.

### Docker Swarm Blueprint

The existing Docker Swarm cluster blueprint remains mostly unchanged except for locating the cluster in the `10.100.100.0/24` network, as well as having an interface to the `10.100.101.0/24` network.  In the case of an Openstack platform, this would mean defining a Port node on the existing Kubernetes network, and defining a relationship between the instance and port.

### Service Blueprint

The `service` blueprint has the responsibility to deploy the microservices (i.e. VNFs) to both clusters and configuring them properly.  It does this by exploiting a plugin that "proxies" the Kubernetes and Swarm blueprints as described earlier, and by using the Kubernetes and Swarm plugins to do the actual deployment.

#### Orchestrating The Quagga Container on Kubernetes

Quagga is deployed on Kubernetes using a native Kubernetes descriptor.  For this example Quagga was only deployed to serve simple static routes.  As is typical with the Kubernetes plugin, a Kubernetes descriptor is referred to in the blueprint possibly with some overrides and environment variables that the container(s) can use to self configure.  In this case, the Quagga router is seeded with some static routes created by examining the outputs of the deployment proxy for the Kubernetes deployment, and passing them in the environment to the container.

```yaml
  quagga:
    type: cloudify.kubernetes.Microservice
    properties:
      name: nginx
      ssh_username: ubuntu
      ssh_keyfilename: /root/.ssh/agent_key.pem
      config_files:
        - file: resources/kubernetes/pod.yaml
        - file: resources/kubernetes/service.yaml
      env:
        ROUTES: [ {concat: [ get_property: [ kubernetes_proxy, vm_info, apache_subnet ], " dev eth1" ]} ]
    relationships:
      - type: cloudify.kubernetes.relationships.connected_to_master
        target: kubernetes_proxy
```

The Quagga container deployment descriptor is simple, but note that it must be run with privileged access and use the host network stack:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: quagga
spec:
  replicas: 1
  selector:
    app: quagga
  template:
    metadata:
      name: quagga
      labels:
        app: quagga
    spec:
      hostNetwork: true
      containers:
      - name: quagga
        image: dfilppi/quagga
        workingDir: /
        command: ["bash","start.sh"]
        ports:
        - containerPort: 2601
          hostIP: 0.0.0.0
        securityContext:
          _privileged: true_
```

The start script in the container takes care of populating the static routes and starting the router.  As a side note, it is assumed that ip forwarding is turned in the instance.

#### Deploying and Configuring The Nginx Container

The last piece of the puzzle is deploying the Nginx container.  The only significant configuration step is populating the load balance host list.  The approach is similar to that used in Kubernetes (passing config in environment variables), but the plugin is different.  Whereas the Kubernetes plugin uses native Kubernetes descriptors, the current version of the Swarm plugin does not handle the equivalent for Swarm (Docker Compose).  Instead, the configuration in the blueprint is more explicit, with the plugin defining `cloudify.swarm.Microservice`,`cloudify.swarm.Container`, and `cloudify.swarm.Port` types.  The microservice is loaded into the defined container, and the ports are exposed via relationships.  In this case, the container accepts the environment config and image reference.

```yaml
  nginx_container:
    type: cloudify.swarm.Container
    properties:
      image: dfilppi/nginx2
      entry_point: start.sh
      env:
        SERVERS: {get_attribute: [kubernetes_proxy,vm_info,ips]}
```

Note how the `SERVERS` definition connects the dynamic outputs of the Kubernetes blueprint to the container configuration in the Swarm cluster.  That pattern of proxied, hybrid orchestration has application far from this esoteric use case.  It is similar to the approach taken in a previous orchestration that demonstrated scaling in Kubernetes triggered by activity on cloud [vms](http://getcloudify.org/2016/03/24/openstack-scaling-kubernetes-microservices-linux-containers-cloud-TOSCA-orchestration.html).