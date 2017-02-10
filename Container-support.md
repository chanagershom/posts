### A Guide To Cloudify Docker Container Support

## Overview

Cloudify supports integrations with Docker and Docker-based container managers, including Docker, Docker Swarm, Docker Compose, Kubernetes, and Apache Mesos.  At a minimum, Cloudify supports the creation, scaling, and healing of the platforms themselves.  For Kubernetes and Docker Swarm, service orchestration is also supported.  When orchestrating other orchestrators (e.g. Kubernetes, Swarm, Mesos), the Cloudify philosophy is to lightly integrate so that native descriptors can be used if desired.  Options are also provided to use TOSCA based configuration for services, but support is more limited than native descriptors.

## Docker Plugin

The [Docker plugin](https://github.com/cloudify-cosmo/cloudify-docker-plugin) is a Cloudify plugin that defines a single type: `cloudify.docker.Container`.  The plugin is compatible with Docker 1.0 (API version 1.12) and relies on the [docker-py](https://github.com/docker/docker-py) library.  The plugin executes on a computer host that has Docker pre-installed.  

### Types

#### cloudify.docker.Container

##### Properties:

* `image` A dict describing a docker image. To import an image from a tarball
          use the src key. The value will be an absolute path or URL. If pulling
          an image from docker hub, do not use src. The key is repository. The value is that
          repository name. You may additionally specify the tag, if none is given,
          latest is assumed.
* `name` The name of the Docker container. This will also be the host name in Docker
          host config.
* `use_external_resource` Boolean indicating whether the container already exists or not.

##### Interfaces:

The `cloudify.interfaces.lifecycle` interface is implemented, and supports the following function parameters

* `create` inputs:
 * `params` A dict of parameters allowed by docker-py to the
                create_container function
* `start` inputs:
 * `params` A dictionary of parameters allowed by docker-py to the
                start function
 * `processes_to_wait_for` A list of processes to verify are active on the container
                before completing the start operation. If all processes are not active
                the function will be retried.
 * `retry_interval` Before finishing start checks to see that all processes
                on the container a                A dictionary of parameters allowed by docker-py to the
                stop function.re ready. This is the interval between
                checks.
* `stop` inputs:
 * `params` A dictionary of parameters allowed by docker-py to the
                stop function.
 * `retry_interval` If Exited is not in the container status, then the plugin will retry. This is
                the number of seconds between retries.
* `delete` inputs:
 * `params` A dictionary of parameters allowed by docker-py to the
                remove_container function.

## Docker Swarm Blueprint

The [Docker Swarm blueprint](https://github.com/cloudify-examples/docker-swarm-blueprint) creates and manages a Docker Swarm cluster on Openstack.  There are 3 blueprints, with slightly different use cases:

* swarm-local-blueprint.yaml : a cfy local blueprint that orchestrates setup and teardown of the cluster without a manager
* swarm-openstack-blueprint.yaml : an Openstack blueprint that orchestrates setup and teardown of the cluster with a manager
* swarm-scale-blueprint.yaml : an Openstack blueprint that orchestrates setup, teardown, autohealing, and autoscaling of the cluster

### Prerequisites

These blueprints have only been tested against an Ubuntu 14.04 image with 2GB of RAM. The image used must be pre-installed with Docker 1.12. Any image used should have passwordless ssh, and passwordless sudo with requiretty false or commented out in sudoers. Also required is an Openstack cloud environment. The blueprints were tested on Openstack Kilo.

### Usage

#### swarm-local-blueprint.yaml
##### Overview
The `swarm-local` blueprint is intended to be run using the [cfy local](http://docs.getcloudify.org/3.4.1/cli/local/) CLI command.  As such, no manager is necessary.  The blueprint starts a two node Swarm cluster and related networking infrastructure in Openstack.
##### Inputs
* `image` The Openstack image id.  This image will be used for both master and worker nodes.  This image must be prepared with Docker 1.12, as well as support passwordless ssh, passwordless sudo, and passwordless sudo over ssh.  Only Ubuntu 14.04 images have been tested.
* `flavor` The Openstack flavor id.  This flavor will be used for both master and worker nodes.  2 GB RAM flavors and 20 GB disk are adequate.  Flavor size will vary based on application needs.
* `ssh_user` This blueprint uses the [Fabric plugin](http://docs.getcloudify.org/3.4.1/plugins/fabric/) and so requires ssh credentials.
* `ssh_keyname` The Openstack ssh key to attach to the compute nodes (both master and worker).
* `ssh_keyfile` This blueprint uses the [Fabric plugin](http://docs.getcloudify.org/3.4.1/plugins/fabric/) and so requires ssh credentials.

##### Other Configuration
The blueprint contains a `dsl_definitions` block to specify the Openstack credentials:  
* `username` The Openstack user name
* `password` The Openstack password
* `tenant_name` The Openstack tenant
* `auth_url` The Openstack Keystone URL

#### swarm-openstack-blueprint.yaml

#### swarm-scale-blueprint.yaml

## Docker Swarm Plugin

The [Docker Swarm Plugin](https://github.com/cloudify-examples/cloudify-swarm-plugin) provides support for deploying services onto [Docker Swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) clusters, as well as support for [Docker Compose](https://docs.docker.com/compose/overview/).

## Kubernetes Blueprint

## Kubernetes Plugin

The [Kubernetes Plugin](https://github.com/cloudify-examples/cloudify-kubernetes-plugin) provides support for deploying services on [Kubernetes](https://kubernetes.io/docs/) clusters.

## Kubernetes Blueprint

The [Kubernetes Blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint) creates and manages a [Kubernetes](https://kubernetes.io/docs/) cluster on Openstack and Amazon EC2.

## Mesos Blueprint

The [Mesos](TBD) creates and manages [Mesos](http://mesos.apache.org/) clusters on Openstack.
