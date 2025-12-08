# SealedSecrets

SealedSecrets provides a way to encrypt Kubernetes Secrets so they can be safely stored in Git repositories (GitOps workflow). Unlike regular Secrets which are only base64 encoded, SealedSecrets are encrypted and can only be decrypted by the controller running in the cluster.

## How It Works

1. **Sealed Secrets Controller**: Runs in the cluster and holds the private encryption key
2. **kubeseal CLI**: Encrypts Secrets using the cluster's public key
3. **SealedSecret Custom Resource**: Encrypted secret stored in Git
4. **Automatic Decryption**: Controller decrypts SealedSecret and creates regular Secret in the cluster

## Usage Workflow

### 1. Create a Regular Secret YAML

Create your secret as usual (but don't apply it to the cluster):

```yaml
# example-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: wordpressdb
  namespace: config-example
type: Opaque
stringData:
  host: localhost
  port: "3306"
  username: wordpress
  password: mysecretpass
```

### 2. Encrypt the Secret with kubeseal

Convert the regular Secret to a SealedSecret:

```bash
kubeseal -o yaml < example-secret.yaml > sealedsecret.yaml
```

### 3. Commit SealedSecret to Git

The generated `sealedsecret.yaml` is safe to commit:

```bash
git add sealedsecret.yaml
git commit -m "Add encrypted database credentials"
git push
```

### 4. Apply SealedSecret to Cluster

```bash
kubectl apply -f sealedsecret.yaml
```

The controller will automatically decrypt it and create a regular Secret:

```bash
# View the SealedSecret (encrypted)
kubectl get sealedsecret wordpressdb -n config-example -o yaml

# View the decrypted Secret
kubectl get secret wordpressdb -n config-example -o yaml
```

## Scopes

SealedSecrets support three encryption scopes:

### Strict (default)
- Most secure
- Sealed for specific name and namespace
- Cannot be renamed or moved to different namespace

```bash
kubeseal --scope strict < secret.yaml > sealed-strict.yaml
```

### Namespace-wide
- Can be renamed within the same namespace
- Cannot be moved to different namespace

```bash
kubeseal --scope namespace-wide < secret.yaml > sealed-ns.yaml
```

### Cluster-wide
- Least secure
- Can be renamed and moved to any namespace
- Useful for shared secrets

```bash
kubeseal --scope cluster-wide < secret.yaml > sealed-cluster.yaml
```

## Best Practices

1. **Never commit unencrypted secrets** to Git
2. **Backup sealing keys** regularly and store securely
3. **Use strict scope** unless you have specific reasons for namespace/cluster-wide (default)
4. **Rotate secrets regularly** even though they're encrypted (default)
5. **Limit access** to kubeseal certificates and sealing keys

## Additional Resources

- [Official Documentation](https://github.com/bitnami-labs/sealed-secrets)
- [Security Model](https://github.com/bitnami-labs/sealed-secrets#security)
- [FAQ](https://github.com/bitnami-labs/sealed-secrets#faq)
