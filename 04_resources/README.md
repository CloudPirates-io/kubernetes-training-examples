# Resource Management

Requests and limits tell Kubernetes how much CPU and memory a container needs and may use.

## Concepts

- **Requests** - used for scheduling; the node must have this much available.
- **Limits** - enforced at runtime. CPU over the limit is throttled; memory over the limit gets the container OOM-killed.
- **QoS classes** (derived from requests/limits, see `qos/`):
  - **Guaranteed** - requests equal limits for all containers; evicted last.
  - **Burstable** - at least one request or limit set.
  - **BestEffort** - nothing set; evicted first.
- **ResourceQuota** (`quota.yaml`) - caps total resource usage per namespace.

## Units

- **CPU** - `1` CPU = 1 vCPU/vCore/hyperthread; `1000m` (millicores) = `1`.
- **Memory** - bytes with suffixes: `M`, `G` (powers of 10) or `Mi`, `Gi` (powers of 2), e.g. `128Mi`.

## Best practices

- **Requests** = what the container actually uses.
- **Limits** = requests + headroom for spikes.
- Always set memory limits. Memory cannot be throttled, so a missing limit risks OOM kills on the whole node.

## Directory Layout

- `01-full-example.yaml` - requests and limits combined
- `cpu/` - CPU behaviour and node limits
- `memory/` - exceeding memory limits, node limits
- `qos/` - one example per QoS class (namespace `qos-example`)
