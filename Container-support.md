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

##### Overview

The [swarm-openstack-blueprint.yaml](https://github.com/cloudify-examples/docker-swarm-blueprint/blob/master/swarm-openstack-blueprint.yaml) is a Cloudify manager hosted blueprint that starts a Swarm cluster and related networking infrastucture.

##### Inputs
* `image` The Openstack image id.  This image will be used for both master and worker nodes.  This image must be prepared with Docker 1.12, as well as support passwordless ssh, passwordless sudo, and passwordless sudo over ssh.  Only Ubuntu 14.04 images have been tested.
* `flavor` The Openstack flavor id.  This flavor will be used for both master and worker nodes.  2 GB RAM flavors and 20 GB disk are adequate.  Flavor size will vary based on application needs.
* `ssh_user` This blueprint uses the [Fabric plugin](http://docs.getcloudify.org/3.4.1/plugins/fabric/) and so requires ssh credentials.
* `agent_user` The user for the image.

##### Outputs
* `swarm-info` which is a dict with two keys:
 * `manager_ip` the public IP address allocated to the Swarm manager
 * `manager_port` the port the manager listens on

#### swarm-scale-blueprint.yaml

##### Overview
The [swarm-scale-blueprint.yaml](https://github.com/cloudify-examples/docker-swarm-blueprint/blob/master/swarm-openstack-blueprint.yaml) is a Cloudify manager hosted blueprint that starts a Swarm cluster and related networking infrastucture.  It installs metrics collectors on worker nodes, and defines scaling and healing groups for cluster high availability.

##### Inputs
* `image` The Openstack image id.  This image will be used for both master and worker nodes.  This image must be prepared with Docker 1.12, as well as support passwordless ssh, passwordless sudo, and passwordless sudo over ssh.  Only Ubuntu 14.04 images have been tested.
* `flavor` The Openstack flavor id.  This flavor will be used for both master and worker nodes.  2 GB RAM flavors and 20 GB disk are adequate.  Flavor size will vary based on application needs.
* `ssh_user` This blueprint uses the [Fabric plugin](http://docs.getcloudify.org/3.4.1/plugins/fabric/) and so requires ssh credentials.
* `agent_user` The user for the image.

##### Outputs
* `swarm-info` which is a dict with two keys:
 * `manager_ip` the public IP address allocated to the Swarm manager
 * `manager_port` the port the manager listens on

## Docker Swarm Plugin

The [Docker Swarm Plugin](https://github.com/cloudify-examples/cloudify-swarm-plugin) provides support for deploying services onto [Docker Swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/) clusters, as well as support for [Docker Compose](https://docs.docker.com/compose/overview/).

### Types

#### cloudify.swarm.Manager
##### Overview
A type that represents a Swarm manager not managed by Cloudify.  If a Cloudify managed manager is used, the [Cloudify proxy plugin](https://github.com/cloudify-examples/cloudify-proxy-plugin) should be used instead.
##### Properties
* `ip` The IPV4 address of the Swarm manager
* `port` The port the manager REST API is listening on (default 2375)
* `ssh_user` An ssh user for operations that require ssh (Docker Compose)
* `ssh_keyfile` An ssh private key for operations that require ssh (Docker Compose)

#### cloudify.swarm.Microservice
##### Overview
The `cloudify.swarm.Microservice` type represents a Docker Swarm service.  It can be configured to use TOSCA-style properties or point to an external Swarm yaml descriptor.  Note that the source project has an example of usage.

##### Properties
* `compose_file` The path to a Docker compose descriptor file.  If set, all other properties are ignored.
* all other properties are translated into the Docker REST [service create ](https://docs.docker.com/v1.12/engine/reference/api/docker_remote_api_v1.24#create-a-service) API call.  Properties in the blueprint are encoded with underscores between words (e.g. `log_driver`) and converted internally to the REST API body camel case (e.g. `LogDriver`).  See comments in the [plugin.yaml](https://github.com/cloudify-examples/cloudify-swarm-plugin/blob/master/plugin.yaml) for an extensive example.

##### Relationships

* `cloudify.swarm.relationships.microservice_contained_in_manager` This relationship connects a Microservice to a manager.  The implementation allows the target to be either a `cloudify.swarm.Manager` type or a `cloudify.nodes.DeploymentProxy` type.

## Kubernetes Cluster Blueprint

The [Kubernetes Cluster Blueprint](https://github.com/cloudify-examples/kubernetes-cluster-blueprint) creates and manages a [Kubernetes](https://kubernetes.io/docs/) cluster on Openstack and Amazon EC2.  It uses the [containerized version of Kubernetes](https://kubernetes.io/docs/getting-started-guides/docker-multinode) to create the cluster.  It also installs the Kubernetes dashboard and the `kubectl` utility on the master.  By default, the blueprint is configured to install on AWS.  To switch to Openstack, edit the [blueprint file](https://github.com/cloudify-examples/kubernetes-cluster-blueprint/blob/master/kubernetes-blueprint.yaml) and comment out the line `- imports/aws/blueprint.yaml`.  Then uncomment the line below.

### Inputs (Common)
* `your_kubernetes_version` The version of Kubernetes to use.  Default = 1.2.1.
* `your_etcd_version` The version of Etcd to use. Default = 2.2.1.
* `your_flannel_version` The version of Flannel to use.  Default = 0.5.5
* `flannel_interface` The interface to bind flannel to.  Default = eth0
* `flannel_ipmasq_flag`  Whether to flannel should use IP Masquerading.  Default = true

### Inputs (AWS)
* `aws_access_key_id` The AWS access key
* `aws_secret_access_key` The AWS secret key
* `ec2_region_name` The EC2 region name.  Default = us-east-1
* `ec2_region_endpoint` The EC2 region. Default = ec2.us-east-1.amazonaws.com

### Inputs (Openstack)
* `keystone_username` Openstack user name
* 'keystone_password` Openstack password
* `keystone_tenant_name` Openstack tenant
* `keystone_url` Openstack authentication URL
* `region` Openstack region (optional)
* `nova_url` Openstack Nova compute API URL (optional)
* `neutron_url` Openstack Neutron network API URL (optional)
* `openstack_management_network_name` The Cloudify management network name (optional)

### Outputs

#### AWS
* A single output `Kubernetes_Dashboard` with a dict value with a single key `url`.  URL uses the floating IP allocated to point to the Kubernetes dashboard.

#### Openstack
* A single output `kubernetes_info` with a dict value with a single key `url`.  URL uses the floating IP allocated to point to the Kubernetes dashboard.

### Other Configuration

To tweak the scaling behavior, the groups are defined in the individual cloud specific imports for [AWS](https://github.com/cloudify-examples/kubernetes-cluster-blueprint/blob/master/imports/aws/blueprint.yaml) and [Openstack](https://github.com/cloudify-examples/kubernetes-cluster-blueprint/blob/master/imports/openstack/blueprint.yaml).  Both sub-blueprints refer to a custom scaling policy [type](https://github.com/cloudify-examples/kubernetes-cluster-blueprint/blob/master/imports/scale.yaml).  The type definition documents how the scaling parameters can be tweaked for desired effects.  The heal group uses the built in [host failure policy](http://docs.getcloudify.org/3.4.1/manager_policies/built-in-policies/) which is triggered by named metrics expiring (60 seconds).

## Kubernetes Plugin

The [Kubernetes Plugin](https://github.com/cloudify-examples/cloudify-kubernetes-plugin) provides support for deploying services on [Kubernetes](https://kubernetes.io/docs/) clusters.


## Mesos Blueprint

The [Mesos](TBD) creates and manages [Mesos](http://mesos.apache.org/) clusters on Openstack.
