## Kubernetes Hosted VNFs, A Simple Example

An important use case for virtual network functions is using container technology rather than OS virtualization. The advantages of containerization include agility, performance, and density/efficiency.  Kubernetes (managing Docker containers) is the leading (and most capable) container management platform today, and the logical platform to use for technical exploration.  In this post, we'll explore a Cloudify orchestrated example of deploying a VNF as a microservice in Kubernetes.

### Summary

The [Quagga](http://www.nongnu.org/quagga) router is software that turns a Linux host into a router.  Any Linux host with IP forwarding capability can function as a router simply by modifying the kernel routing table using standard shell commands (.e.g. ip route).  Quagga builds on this capability and provides dynamic routing capability via various standard routing protocols, including OSPF, RIP, BGP and others.  This post explores the process of containerizing Quagga and deploying it using Kubernetes.  In the project, Cloudify is used to run Kubernetes, and deploys Quagga as a microservice using TOSCA modeling.  

