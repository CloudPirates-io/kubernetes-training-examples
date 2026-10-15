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

## Directory Layout

- `01-full-example.yaml` - requests and limits combined
- `cpu/` - CPU behaviour and node limits
- `memory/` - exceeding memory limits, node limits
- `qos/` - one example per QoS class (namespace `qos-example`)
