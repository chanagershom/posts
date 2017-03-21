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

This descriptor is consumed by the host pool service and represents the initial configuration.  Bear in mind that the configuration isn't static, and can be amended and updated via REST API.  In a blueprint, utilizing the host pool plugin, a hosts can be requested from the pool.  The resulting host pool can be treated like any other compute resource for the purposes of orchestration.

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

