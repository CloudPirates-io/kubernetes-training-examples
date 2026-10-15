# Configuration and Secrets

Container images should be environment-independent. Configuration is injected at runtime.

## Resources

| Resource | Purpose | Directory |
|----------|---------|-----------|
| **ConfigMap** | Non-sensitive config | `configmaps/` |
| **Secret** | Sensitive data (base64 encoded, *not* encrypted by default) | `secrets/` |
| **SealedSecret** | Encrypted Secret that is safe to commit to Git | `sealedsecrets/` |
| **PodDisruptionBudget** | Keeps a minimum number of Pods available during voluntary disruptions such as node drains | `pdb/` |

## Ways to consume ConfigMaps and Secrets

- A single key as an environment variable
- All keys as environment variables (`envFrom`)
- Mounted as files via a volume (updated automatically for volumes, not for env vars)

Secrets can be written with `data` (base64) or `stringData` (plain text, encoded by the API server).

## PodDisruptionBudgets

Use either `minAvailable` or `maxUnavailable` to limit how many Pods may be evicted at once.

## Namespaces

`configmaps/` and `secrets/` use `config-management`; `pdb/` uses `pdb-examples`.
