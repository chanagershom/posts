<img src="https://github.com/dfilppi/posts/blob/master/images/kub-bare/bare.png" width="320px" align="center"/>

## Orchestrating Kubernetes on Bare Metal

Linux container technology permits the subdivision of compute resources in ways similar to virtualization, but without the overhead of managing separate copies of the operating system in each instance/container.  To maximize this effect, containers are frequently run on "bare metal" (non-virtualized operating systems).  This post explains the approach Cloudify takes to orchestrate the leading container management system (Kubernetes) on bare metal.

### Orchestrating Bare Metal with Cloudify

In order to approach the topic of bare metal orchestration of Kubernetes, some background in Cloudify orchestration of bare metal is needed.  At it's core, Cloudify is a distributed task execution engine that is ambivalent about what the tasks are ultimately doing.  A blueprint might describe a node that operates a REST API that talks to a well known cloud (e.g. AWS), or a node that copies files, or any other programming task you might imagine.  In the case of compute resources, blueprints typically describe a unit of code that returns an IP address.  An alternative approach is to simply configure the blueprint node with an existing IP address.  This is the foundation for the [host pool](https://github.com/cloudify-cosmo/cloudify-host-pool-plugin) plugin.

#### The Host Pool Plugin ( and service )

The [host pool](https://github.com/cloudify-cosmo/cloudify-host-pool-plugin) plugin and [service](https://github.com/cloudify-cosmo/cloudify-host-pool-service) manages pools of IP address for Cloudify blueprints.  Hosts and their IP addresses can be tagged and filtered so that a blueprint might ask for an IP address to a machine with certain arbitrary characteristics (e.g. "ubuntu" or "large").  For example, a group of physical hosts can be described like so:

```yaml

default:
  os: linux
  credentials:
    username: ubuntu
  endpoint:
    port: 22
    protocol: ssh

hosts:
  - name: server0
    credentials:
      key_file: keys/key.pem
      username: centos
    endpoint:
      ip: 52.29.45.36
    tags:
      - centos
      - large
    
  - name: server1
    credentials:
      key_file: keys/key.pem
    endpoint:
      ip: 52.58.54.251
    tags:
      - ubuntu
      - medium

  - name: server2
    credentials:
      key_file: keys/key.pem
    endpoint:
      ip: 52.58.68.85
    tags:
      - ubuntu
      - medium
```

This descriptor is consumed by the host pool service and represents the initial configuration.  Bear in mind that the configuration isn't static, and can be amended and updated via REST API.  In a blueprint, utilizing the host pool plugin, a host can be requested from the pool.  The resulting host pool can be treated like any other compute resource for the purposes of orchestration.

```yaml
node_templates:
...
  node1:
    type: cloudify.hostpool.nodes.LinuxHost
    properties:
      filters:
        tags:
          - ubuntu
          - large
```

#### Honorable Mention: The Netconf Plugin

While it is likely in a non-virtualized installation of Kubernetes that networking will be set up manually, for completeness it is necessary to including a mention of the Cloudify [Netconf Plugin](https://github.com/cloudify-cosmo/cloudify-netconf-plugin).  This plugin provides bare metal orchestration capability in the networking domain (for devices that support [Netconf](https://tools.ietf.org/html/rfc6241)).  Of related usefulness is the [yttc](https://github.com/cloudify-cosmo/yttc) tool that converts YANG syntax to TOSCA types for inclusion in blueprints.

### Kubernetes On Bare Metal

Once the groundwork of understanding and configuring a Host Pool service is complete, the orchestration of Kubernetes itself is straightforward.  An existing Kubernetes [blueprint](https://github.com/cloudify-examples/simple-kubernetes-blueprint/blob/master/openstack-blueprint.yaml) can be adapted to operate on the bare metal infrastructure.  In the case of the referenced blueprint (originally targeting Openstack), the exercise starts with including a reference to the Host Pool plugin and updating the compute nodes in the blueprint.  Since we aren't orchestrating networking in this example, we can remove all relationships, and then replacing Openstack related config with Host Pool equivalents.  The result is something like this (using the sample service filters from above):

__BEFORE__
```yaml
kubernetes_master_host:
    type: cloudify.openstack.nodes.Server
    properties:
      agent_config:
        <<: *agent_config
      image: { get_input: image }
      flavor: { get_input: flavor }
    relationships:
      - target: kubernetes_security_group
        type: cloudify.openstack.server_connected_to_security_group
      - type: cloudify.openstack.server_connected_to_floating_ip
        target: kubernetes_master_ip
```

__AFTER__
```yaml
kubernetes_master_host:
    type: cloudify.hostpool.nodes.LinuxHost
    properties:
      agent_config:
        <<: *agent_config
      filters:
        tags:
          - large
          - centos
```

The remainder of the effort is largely removing all networking configuration.  Autoscaling and healing will function as expected, assuming the host pool has sufficient hosts defined.  Note also that the host pool configuration isn't fixed, and that additional machines can be made available by updating the host pool service via it's REST API.

### Conclusion

This post explained how Kubernetes can be managed by Cloudify on bare metal by a simple modification of already available blueprints.  This is a good example of the power of the compute abstraction in TOSCA modeling, made possible by the fact that both the pre-conversion Openstack `Server` type and the post-conversion Host Pool `Host` (or in this case the derived `LinuxHost` type are both derived ultimately from the `cloudify.nodes.Compute` type, which makes them easily interchangeable in blueprints.  It also demonstrates how, and why, Cloudify orchestration is not limited to clouds or virtualization.
