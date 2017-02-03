### A Guide To Cloudify Linux Container Support

## Overview

Cloudify supports integrations with Docker and Docker-based container managers, including Docker, Docker Swarm, Docker Compose, Kubernetes, and Apache Mesos.  At a minimum, Cloudify supports the creation, scaling, and healing of the platforms themselves.  For Kubernetes and Docker Swarm, service orchestration is also supported.  When orchestrating other orchestrators (e.g. Kubernetes, Swarm, Mesos), the Cloudify philosophy is to lightly integrate so that native descriptors can be used if desired.  Options are also provided to use TOSCA based configuration for services, but support is more limited than native descriptors.