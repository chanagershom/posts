<img src="https://github.com/dfilppi/posts/blob/master/images/bmc-plugin/oraclebmc.png" />

Late last year, Oracle [announced](https://blogs.oracle.com/cloud/entry/oracle_bare_metal_cloud_services) the availability of their Bare Metal Cloud IAAS to the world.  The Bare Metal Cloud (BMC) combines physical instances combined with a fully virtualized networking infrastructure that emulates a traditional datacenter, but with the convenience and agility that infrastructure as a service provides.  With non-virtualized compute and microsecond inter-host network latency, the BMC is particularly suited to demanding high performance workloads.  For the latest on the bare metal cloud, see the Oracle [website](https://cloud.oracle.com/en_US/bare-metal).  This post covers the Cloudify integration with the Oracle BMC.

## The Cloudify Oracle BMC Plugin

Like all IAAS plugins for Cloudify, the BMC plugin maps nouns in the underlying IAAS API to TOSCA types, and the verbs get mapped to orchestrator lifecycle events (e.g. "create" and "configure").  The current version of the BMC plugin only addresses compute and networking domains of the BMC [SDK](https://oracle-bare-metal-cloud-services-python-sdk.readthedocs.io/en/latest/).  The plugin provides the ability to orchestrate instances (both bare metal and virtual), along with related networking components such as networks, subnets, security, and internet gateways.


<img src="https://github.com/dfilppi/posts/blob/master/images/bmc-plugin/bmc-plugin-components.png" />


The following components and relationships are supported:

* #### `cloudify.oraclebmc.nodes.Instance`
Represents a compute node, either bare metal or virtual (based on the "instance_shape") property.  It accepts configuration properties such as `image_id`, `instance_shape`, `compartment_id`, and `availability_domain`.  After the `install` workflow is complete, the instance will have attributes `public_ip` and `private_ip`.  It is connected to a subnet (described below), via the `cloudify.oraclebmc.relationships.instance_connected_to_subnet` relationship.

* #### `cloudify.oraclebmc.nodes.VCN`
This node type represents a network.  The main unique configuration property is the network `cidr_block`, from which subnets are carved out.

* #### `cloudify.oraclebmc.nodes.Subnet`
This node type represents a subnet network.  This node is configured with a `cidr_block` and `security_list`, among other properties.  `cidr_block` represents the portion of the VNC network CIDR block, and the `security_list` is a list of firewall rules.  It is associated with a target network via the `cloudify.oraclebmc.relationships.subnet_in_network` relationship.

* #### `cloudify.oraclebmc.nodes.Gateway`
This node represents an internet gateway.  It is configured with a list of routing rules, which amounts to an internet firewall.  It is associated with a target network via the `cloudify.oraclebmc.relationships.gateway_connected_to_network` relationship.

An simple single instance example blueprint can be found in the plugin [examples](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin/blob/master/examples/blueprint/test.yaml) directory.  Bear in mind that it, and the other example mentioned later, rely on images that have `firewalld` disabled. 

```yaml

node_templates:

  server:
    type: cloudify.oraclebmc.nodes.Instance
    properties:
      install_agent: false
      bmc_config: *bmc_config
      public_key_file: 'somekey.pub'
...
```

A more elaborate example blueprint can be found in the Cloudify Examples [repo](https://github.com/cloudify-examples/simple-kubernetes-blueprint/blob/master/bmc-blueprint.yaml), which starts an autoscaling/autohealing Kubernetes cluster.

### Limitations

When used with the Cloudify Manager, the plugin must be installed via the `cfy plugins upload` method.  A `wgn` package is included in the [repo](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin).  The plugin currently only addresses compute and networking aspects of the cloud API.  Future revisions will expand support to other aspects such as storage.

## The Cloudify Oracle BMC Manager Blueprint

<img src="http://docs.getcloudify.org/3.4.1/images/architecture/cloudify_advanced_architecture.png"/>

In order to fully support Cloudify orchestration capabilities, a Cloudify manager is needed.  In the [`examples/manager`](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin/tree/master/examples/manager) directory, there is a Cloudify manager [blueprint](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin/blob/master/examples/manager/oracle-bmc-manager-blueprint.yaml) that support the Oracle Bare Metal Cloud.  This blueprint is used to bootstrap a Cloudify manager in a similar fashion to manager blueprints for [other](https://github.com/cloudify-cosmo/cloudify-manager-blueprints) clouds.  The blueprint [inputs](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin/blob/master/examples/manager/oracle-bmc-manager-blueprint-inputs.yaml) file has description of the settings needed.

### Prerequisites.

The image used must be a Centos 7 image on an instance shape with at least 8GB of RAM.  The image must have `firewalld` (enabled by default) _disabled_.  You'll need Oracle BMC API credentials as well.  The blueprints directory contains the file `oracle--agent.tar.gz`.  After manager bootstrap, this file should be added to the `/opt/manager/resources/packages/agents` directory, if you'll be running agents on Oracle Linux machines.

### Bootstrapping

The bootstrapping process is no different from the standard [process](http://docs.getcloudify.org/3.4.1/cli/bootstrap).  First you'll need to get the Cloudify [CLI](http://getcloudify.org/downloads/get_cloudify.html) and install it on a machine with internet access to the BMC API.  Then just:

* `cfy bootstrap --install-plugins -p oracle-bmc-manager-blueprint.yaml -i oracle-bmc-manager-blueprint-inputs.yaml`

After the bootstrap is complete, you can upload blueprints and have full Cloudify manager functionality.

## Conclusion

Oracle has bridged the gap between conventional data centers and the Cloud by providing an infrastructure as a service interface to bare metal servers, while retaining the flexibility of virtual networking.  Now this powerful environment has been added to the ever growing Cloudify family of supported clouds.  As always, the code and examples are available on [github](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin).
