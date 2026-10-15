# Workload Resources

Pods are rarely created directly. Workload controllers create and manage them, and replace them when they fail.

| Resource | Use for | File |
|----------|---------|------|
| **ReplicaSet** | Keeping N identical Pods running | `01-replicaset.yaml` |
| **Deployment** | Stateless apps; manages ReplicaSets, rolling updates, rollbacks | `02-deployment.yaml` |
| **StatefulSet** | Stateful apps needing stable identity and storage (Redis example with `volumeClaimTemplates`) | `03-statefulset.yaml` |
| **DaemonSet** | One Pod per (selected) node, e.g. Prometheus node exporter | `04-daemonset.yaml` |
| **Job** | Run-to-completion batch tasks | `05-job.yaml` |
| **CronJob** | Jobs on a schedule | `06-cronjob.yaml` |

## Key ideas

- Controllers use **label selectors** to find the Pods they own.
- You declare the desired state; controllers continuously reconcile reality towards it.
- A Deployment is the default choice for stateless applications.
