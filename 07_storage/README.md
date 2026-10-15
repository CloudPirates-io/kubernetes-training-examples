# Persistent Storage

Container filesystems are ephemeral. Volumes let data outlive containers, and PersistentVolumes let it outlive Pods.

## Directory Layout

- `special-volumes/`
  - **emptyDir** - scratch space that lives as long as the Pod
  - **hostPath** - a path on the node's filesystem (use sparingly)
- `manual-provisioning/` - an admin creates a **PersistentVolume (PV)**, a user claims it with a **PersistentVolumeClaim (PVC)**
- `dynamic-provisioning/` - a PVC referencing a **StorageClass** automatically gets a PV created

## Concepts

- **PV** - a piece of storage in the cluster.
- **PVC** - a request for storage by a user; binds to a matching PV.
- **StorageClass** - describes a "class" of storage and the provisioner that creates it.
- **Access modes** - `ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`, `ReadWriteOncePod`.
- **Reclaim policy** - `Delete` or `Retain` what happens to the PV when the PVC is removed.

