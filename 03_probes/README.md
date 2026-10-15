# Probes

Probes let the kubelet check the health of a container and react to it.

## Probe types (what is checked)

| Mechanism | Description | Example |
|-----------|-------------|---------|
| **HTTP** | GET request, success on 2xx/3xx | `01-deployment.yaml` |
| **TCP** | Open a socket connection | `02-deployment-tcp-probe.yaml` (Redis) |
| **Exec** | Run a command in the container, success on exit code 0 | `03-deployment-exec-probe.yaml` (PostgreSQL) |

## Probe kinds (what happens on failure)

- **Startup probe** - protects slow-starting apps; other probes are disabled until it succeeds.
- **Readiness probe** - on failure the Pod is removed from Service endpoints, but not restarted.
- **Liveness probe** - on failure the container is restarted.

## Best practices

- Keep liveness probes cheap and independent of external dependencies.
- Use readiness for "can I serve traffic right now?".
- Tune `initialDelaySeconds`, `periodSeconds`, `timeoutSeconds` and `failureThreshold` to the app.
