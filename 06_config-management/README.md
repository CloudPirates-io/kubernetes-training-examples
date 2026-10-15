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

Secrets can be written with `data` (base64) or `stringData` (plain text, encoded by the API server). Inside the container the values are decoded automatically.

```bash
# Encode a value for the data field
echo -n 'admin' | base64           # YWRtaW4=

# Decode a value
echo 'YWRtaW4=' | base64 --decode  # admin
```

Use `-n`, otherwise the trailing newline becomes part of the secret.

## PodDisruptionBudgets

Use either `minAvailable` or `maxUnavailable` to limit how many Pods may be evicted at once.

## Namespaces

`configmaps/` and `secrets/` use `config-management`; `pdb/` uses `pdb-examples`.
