# nomad-shared-mkdir

A dynamic HashiCorp Nomad host volume plugin optimized for self-orchestrating automatic clustered creation and cleanup across native shared network-mounted filesystems.

By default, Nomad requires that Host Volumes are individually registered into the state store per node. When backed by cluster-wide storage (such as an NFS share globally mapped via Ansible onto all clustered agents), relying on Nomad to dynamically place allocations effectively forces administrators to awkwardly build secondary deployment scripts repeatedly mapping matching folder IDs.

`nomad-shared-mkdir` organically solves this constraint natively within Nomad's Host Volume API abstraction:
- **Instant Cluster Integration:** Dynamically executing `nomad volume create` immediately intercepts the pipeline, spawning automatic UUID assignments safely into neighboring clustered workers sequentially so that allocations schedule painlessly.
- **Cascading Garbage Collection:** Calling `nomad volume delete -type=host <ID>` seamlessly forces local host files removal while actively commanding Nomad Server to atomically strike identical network mapped copies safely without launching into infinite recursion storms.
- **Network Extensibility:** Natively passes down standard IO permissions granting complete compatibility directly out of the box with standard container deployments like `redis` unprivileged endpoints perfectly mirroring original NFS maps.
- **Self-Healing Reconciliation:** On each `fingerprint`, the plugin reconciles any volumes that exist elsewhere in the cluster but are missing a local registration — re-registering them against the shared NFS path. This recovers nodes that were down or absent when a volume was originally created. The reconciliation runs only once per boot or Nomad service restart (guarded by a marker keyed to the boot ID and the Nomad agent's lifetime), so repeated fingerprint calls are cheap no-ops.

## How It Works

The plugin runs three background reconciliation flows, all decoupled from the foreground request so Nomad's `stdout` contract is never blocked:

| Operation | Trigger | Behavior |
| --- | --- | --- |
| `create` | `nomad volume create` | Creates the local directory (`mkdir -p`, mode `0777`) and fans out registration to every other `ready` node pointing at the same shared path. |
| `delete` | `nomad volume delete -type=host <ID>` | Removes the local directory and deregisters all sibling registrations sharing the same volume name, guarded against recursion. |
| `fingerprint` | Periodic, by the Nomad client | Once per boot/agent lifetime, registers any cluster volumes that are missing a registration on the local node. |

Background flows log to `/tmp` (`nomad-shared-mkdir-sync.log` for the fingerprint sync, `auto-mkdir-*.log` for create/delete). The sync log is rotated at 1 MB, keeping a single `.log.1` copy.

> **Note:** This plugin assumes the volume root (`DHV_VOLUMES_DIR`) is a shared filesystem (e.g. NFS) mounted at the same path on every client. Registrations point at that shared path rather than creating per-node local storage.

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