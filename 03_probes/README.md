# Probes

Probes let the kubelet check the health of a container and react to it.

## Probe types (what is checked)

| Mechanism | Description | Example |
|-----------|-------------|---------|
| **HTTP** | GET request, success on 2xx/3xx | `01-deployment.yaml` |
| **TCP** | Open a socket connection | `02-deployment-tcp-probe.yaml` (Redis) |
| **Exec** | Run a command in the container, success on exit code 0 | `03-deployment-exec-probe.yaml` (PostgreSQL) |

## Probe kinds (what happens on failure)

- **Startup probe** - protects slow-starting apps; liveness and readiness probes are disabled until it succeeds. Size it for the worst case: `failureThreshold * periodSeconds` must be longer than the maximum expected startup time.
- **Readiness probe** - on failure the Pod is removed from Service endpoints, but not restarted.
- **Liveness probe** - on failure the container is restarted.

## Best practices

- A liveness probe is not always necessary: if the app crashes itself (exit code > 0), Kubernetes restarts it anyway.
- Always set a readiness probe. Rolling updates and rollbacks rely on it to know when new Pods are ready.
- Implement dedicated endpoints such as `/livez` and `/readyz`, possibly on a separate port from the app.
- Keep probes cheap, fast and independent of external dependencies.
- Probes are not a replacement for CronJobs - don't run periodic work inside them.
- Tune `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds` and `failureThreshold` to the app.
