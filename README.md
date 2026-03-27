# nomad-shared-mkdir

A dynamic HashiCorp Nomad host volume plugin optimized for self-orchestrating automatic clustered creation and cleanup across native shared network-mounted filesystems.

By default, Nomad requires that Host Volumes are individually registered into the state store per node. When backed by cluster-wide storage (such as an NFS share globally mapped via Ansible onto all clustered agents), relying on Nomad to dynamically place allocations effectively forces administrators to awkwardly build secondary deployment scripts repeatedly mapping matching folder IDs.

`nomad-shared-mkdir` organically solves this constraint natively within Nomad's Host Volume API abstraction:
- **Instant Cluster Integration:** Dynamically executing `nomad volume create` immediately intercepts the pipeline, spawning automatic UUID assignments safely into neighboring clustered workers sequentially so that allocations schedule painlessly.
- **Cascading Garbage Collection:** Calling `nomad volume delete -type=host <ID>` seamlessly forces local host files removal while actively commanding Nomad Server to atomically strike identical network mapped copies safely without launching into infinite recursion storms.
- **Network Extensibility:** Natively passes down standard IO permissions granting complete compatibility directly out of the box with standard container deployments like `redis` unprivileged endpoints perfectly mirroring original NFS maps.

## Installation

Because this script intercepts Dynamic Host Volumes natively, make sure it is directly pushed into `/opt/nomad/host-volume-plugins/` separate from normal CSI/Driver installations which load from standard `/opt/nomad/plugins` to prevent internal log parsing conflicts.

```hcl
# nomad.hcl
client {
    host_volume_plugin_dir = "/opt/nomad/host-volume-plugins"
}
```

```bash
# Push binary locally inside driver configuration
cp nomad-shared-mkdir /opt/nomad/host-volume-plugins/nomad-shared-mkdir
chmod +x /opt/nomad/host-volume-plugins/nomad-shared-mkdir

# Restart your agent
systemctl restart nomad
```

## Example Volume Definition
```hcl
name      = "my-distributed-database"
type      = "host"
plugin_id = "nomad-shared-mkdir"
```

License [MIT](LICENSE)