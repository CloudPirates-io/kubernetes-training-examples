# Kubernetes Basics

The **Pod** is the smallest deployable unit in Kubernetes: one or more containers sharing network and storage. Everything else in Kubernetes builds on Pods.

## Concepts

- **Pod** (`pod.yaml`) - a single container with image, ports and resource requests/limits.
- **Init Containers** (`init-container.yaml`) - run to completion, in order, before the main containers start. Useful for setup, waiting for dependencies, or preparing data.
- **Sidecar Containers** (`sidecar-container.yaml`) - helpers running next to the main container for its whole lifetime (logging, proxies, syncing).
