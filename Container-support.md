### A Guide To Cloudify Docker Container Support

## Overview

Cloudify supports integrations with Docker and Docker-based container managers, including Docker, Docker Swarm, Docker Compose, Kubernetes, and Apache Mesos.  At a minimum, Cloudify supports the creation, scaling, and healing of the platforms themselves.  For Kubernetes and Docker Swarm, service orchestration is also supported.  When orchestrating other orchestrators (e.g. Kubernetes, Swarm, Mesos), the Cloudify philosophy is to lightly integrate so that native descriptors can be used if desired.  Options are also provided to use TOSCA based configuration for services, but support is more limited than native descriptors.

## Docker Plugin

The [Docker plugin](https://github.com/cloudify-cosmo/cloudify-docker-plugin) is a Cloudify plugin that defines a single type: `cloudify.docker.Container`.  The plugin is compatible with Docker 1.0 (API version 1.12).  The plugin executes on a computer host that has Docker pre-installed.  

## Docker Swarm Blueprint

The [Docker Swarm blueprint](https://github.com/cloudify-examples/docker-swarm-blueprint) creates and manages a Docker Swarm cluster on Openstack.  

## Docker Swarm Plugin

The [Docker Swarm Plugin](https://github.com/cloudify-examples/cloudify-swarm-plugin) provides support for deploying services onto [Docker Swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) clusters, as well as support for [Docker Compose](https://docs.docker.com/compose/overview/).
## Kubernetes Blueprint

## Kubernetes Plugin

The [Kubernetes Plugin](https://github.com/cloudify-examples/cloudify-kubernetes-plugin) provides support for deploying services on [Kubernetes](https://kubernetes.io/docs/) clusters.

## Kubernetes Blueprint

The [Kubernetes Blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint) creates and manages a [Kubernetes](https://kubernetes.io/docs/) cluster on Openstack and Amazon EC2.

## Mesos Blueprint

The [Mesos](TBD) creates and manages [Mesos](http://mesos.apache.org/) clusters on Openstack.
