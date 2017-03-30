
Late last year, Oracle [announced](https://blogs.oracle.com/cloud/entry/oracle_bare_metal_cloud_services) the availability of their Bare Metal Cloud IAAS to the world.  The Bare Metal Cloud (BMC) combines physical instances combined with a fully virtualized networking infrastructure that emulates a traditional datacenter, but with the convenience and agility that infrastructure as a service provides.  With non-virtualized compute and microsecond inter-host network latency, the BMC is particularly suited to demanding high performance workloads.  For the latest on the bare metal cloud, see the Oracle [website](https://cloud.oracle.com/en_US/bare-metal).  This post covers the Cloudify integration with the Oracle BMC.

## The Cloudify Oracle BMC Plugin

Like all IAAS plugins for Cloudify, the BMC plugin maps nouns in the underlying IAAS API to TOSCA types, and the verbs get mapped to orchestrator lifecycle events (e.g. "create" and "configure").  The current version of the BMC plugin only addresses compute and networking domains of the BMC [SDK](https://oracle-bare-metal-cloud-services-python-sdk.readthedocs.io/en/latest/).  The plugin provides the ability to orchestrate instances (both bare metal and virtual), along with related networking components such as networks, subnets, security, and internet gateways.

An example blueprint can be found in the Cloudify Examples [repo](https://github.com/cloudify-examples/simple-kubernetes-blueprint/blob/master/bmc-blueprint.yaml).  

## The Cloudify Oracle BMC Manager Blueprint

