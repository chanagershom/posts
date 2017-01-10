## Kubernetes Hosted VNFs, A Simple Example

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
        image: dfilppi/quagga5
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
