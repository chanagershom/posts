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

An example blueprint can be found in the Cloudify Examples [repo](https://github.com/cloudify-examples/simple-kubernetes-blueprint/blob/master/bmc-blueprint.yaml).  

### Limitations

When used with the Cloudify Manager, the plugin must be installed via the `cfy plugins upload` method.  A `wgn` package is included in the [repo](https://github.com/cloudify-incubator/cloudify-oraclebmc-plugin).

## The Cloudify Oracle BMC Manager Blueprint

