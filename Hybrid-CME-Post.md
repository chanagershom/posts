![](/images/hybrid-container-vnf/containerized-hybrid-vnf.png)
## Hybrid VNF Container Orchestration 

The need to orchestrate virtual network functions that run in Linux hosted containers is an emerging challenge in the NFV world.  Driven by the need for high responsiveness and deployment density, as well as a desire to adopt modern microservices architectures, users and vendors are finding containers an appealing prospect.  In this post I address a hybrid VNF orchestration consisting of Kubernetes and Swarm container managers, as well as conventional hosts.  The idea is to explore the challenges and opportunities presented by such a scenario.  The world of containerized VNFs is somewhat limited (to say the least) that will run out of the box in Kubernetes and/or Docker Swarm, so I did this exploration with containerized versions of standard Linux network components, the Quagga router and Nginx configured as a load balancer.

### Target Architecture

![Architecture]()

The architecture is composed of some familiar building blocks; the [Kubernetes blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint), the [Kubernetes plugin](https://github.com/cloudify-examples/cloudify-kubernetes-plugin), the [Docker Swarm Blueprint](https://github.com/cloudify-examples/docker-swarm-blueprint), and the [Deployment Proxy plugin](https://github.com/cloudify-examples/cloudify-proxy-plugin).  In addition, a yet to be published Docker Swarm plugin is used.

The basic idea is to run network traffic through one VNF on Docker Swarm, then through another VNF in Kubernetes.  In this example, traffic is load balanced though an Nginx container on Swarm, which load balances a couple of VMs on the other side of a containerized Quagga instance.

