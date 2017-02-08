### A Guide To Cloudify Docker Container Support

## Overview

Cloudify supports integrations with Docker and Docker-based container managers, including Docker, Docker Swarm, Docker Compose, Kubernetes, and Apache Mesos.  At a minimum, Cloudify supports the creation, scaling, and healing of the platforms themselves.  For Kubernetes and Docker Swarm, service orchestration is also supported.  When orchestrating other orchestrators (e.g. Kubernetes, Swarm, Mesos), the Cloudify philosophy is to lightly integrate so that native descriptors can be used if desired.  Options are also provided to use TOSCA based configuration for services, but support is more limited than native descriptors.

## Docker Plugin

The [Docker plugin](https://github.com/cloudify-cosmo/cloudify-docker-plugin) is a Cloudify plugin that defines a single type: `cloudify.docker.Container`.  The plugin is compatible with Docker 1.0 (API version 1.12).  The plugin executes on a computer host that has Docker pre-installed.  

## Docker Swarm Blueprint

## Docker Swarm Plugin

## Kubernetes Blueprint

## Kubernetes Plugin

## Mesos Blueprint

