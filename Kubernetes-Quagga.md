<div align="center">
![alt](https://github.com/dfilppi/posts/blob/master/images/kub-quagga/quagga.jpg)
</div>
## Orchestrating a Kubernetes Managed Virtual Network Function

An important use case for virtual network functions is using container technology rather than OS virtualization. The advantages of containerization include agility, performance, and density/efficiency.  Kubernetes (managing Docker containers) is the leading (and most capable) container management platform today, and the logical platform to use for technical exploration.  In this post, we'll explore a Cloudify orchestrated example of deploying a VNF as a microservice in Kubernetes.

### Summary

The [Quagga](http://www.nongnu.org/quagga) router is software that turns a Linux host into a router.  Any Linux host with IP forwarding capability can function as a router simply by modifying the kernel routing table using standard shell commands (.e.g. ip route).  Quagga builds on this capability and provides dynamic routing capability via various standard routing protocols, including OSPF, RIP, BGP and others.  This post explores the process of containerizing Quagga and deploying it using Kubernetes.  In the project, Cloudify is used to run Kubernetes, and deploys Quagga as a microservice using TOSCA modeling.  

### Containerization Issues

Docker containers (on Linux) typically run in their own network namespace.  In practical terms, this means they have a network stack (including routing tables) independent from the host network stack.  Generally, this is highly desirable, but in the case of a VNF that emulates a general purpose router, it is not desireable.  Fortunately, Docker containers can be configured to use the host network stack when running in privileged mode.  In the case of Quagga, the practical consequence is that the inter-routing procotols will be visible, and changes to the host routing table will actually cause the node to behave as expected.  Taking this approach sacrifices networking isolation on the target host, but the use case justifies it.  Kubernetes supports the starting of privileged containers, so the pod configuration is straightforward.  The key properties are `hostNetwork` and `securityContext`:

```yaml
    spec:
      hostNetwork: true
      containers:
      - name: quagga
        image: quagga
        workingDir: /root
        command: ["bash","/root/start.sh"]
        ports:
        - containerPort: 80
          hostIP: 0.0.0.0
        securityContext:
          privileged: true
        env:
         - name: INTERFACES
           value: eth0,eth1
         - name: ROUTES
           value: 10.0.0.0/24 172.16.0.1,10.10.0.0/24 172.16.0.1
``` 

Beyond networking configuration, the initial configuration of static routes needs to be addressed.  Note the `command` property in the preceding excerpt. This identifies the `start.sh` script as the startup command for the container.  This example uses environment variables, defined in the `env` section, to configure initial static routes by updating the Quagga configuration file and starting the Quagga service.

```bash
IFS=","
for I in $INTERFACES
do
  echo "interface $I" >> /etc/quagga/zebra.conf
done
for R in $ROUTES
do
  echo "ip route $R" >> /etc/quagga/zebra.conf
done
service quagga start
tail -f /dev/null
```

After the script runs, Quagga modifies the host routing tables.  This example only addressed static routing, which is the basis for the dynamic routing protocol implementation.

### Placement

A multi-node Kubernetes deployment is likely to want to constrain the location of the router in the cluster to access multiple interfaces.  Kubernetes permits the labeling of nodes in a cluster with arbitrary values, and then requesting the placement algorithm to filter candidates based on the labels.  This is one area Cloudify can provide value as part of the Kubernetes installation.  See the [Kubernetes example blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint) for a sample Cloudify orchestration.  A simple extension to the schema and some additional logic can provide the needed functionality.  The addition of a `labels` input to the `start.py` script for the kubernetes node provides the labels:

```yaml
      cloudify.interfaces.lifecycle:
        start:
          implementation: scripts/kubernetes/node/start.py
          inputs:
            <<: *kubernetes_environment
            # ADDED LABELS INPUT
            labels:
              - role: router
```

Naturally, the `start.py` script needs to be modified to add the `--labels` parameter for the `hyperkube` startup.  Once done, Kubernetes is ready to filter once started.

### The Router Microservice Definition

A separate Cloudify blueprint from the blueprint that started Kubernetes can be used to model the Quagga router.  The blueprint is small enough to include:

```yaml
node_templates:

  kubernetes_proxy:
    type: cloudify.nodes.DeploymentProxy
    properties:
      inherit_outputs:
        - 'kubernetes_info'
....

  quagga:
    type: cloudify.kubernetes.Microservice
    properties:
      name: nginx
      ssh_username: ubuntu
      ssh_keyfilename: /root/.ssh/agent_key.pem
      config_files:
        - file: resources/kubernetes/pod.yaml
    relationships:
      - type: cloudify.kubernetes.relationships.connected_to_master
        target: kubernetes_proxy
```

Since no overrides are specified for the microservice, the Cloudify blueprint is fairly trivial.  It identifies the Kubernetes master node and `pod.yaml` file (excerpt at the beginning of the post), as well as ssh information for logging into the Kubernetes.  The reason the login information is needed is this sample implementation relies on the `kubectl` command line tool on the master.  The only additional modification needed for placement is to the `pod.yaml` file to target the router node:

```yaml
    spec:
      hostNetwork: true
      # ADD NODE SELECTOR
      nodeSelector:
        role: router
      containers:
      - name: quagga
        image: quagga
        workingDir: /root
....
```

## Conclusion

Kubernetes, Docker, and Cloudify work together seamlessly to create a containerized VNF platform that can deliver high availability, high performance, and high density, while retaining the benefits of a micrservices architecture.