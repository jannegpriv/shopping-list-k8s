# Shopping List Kubernetes Configuration

This repository contains the Kubernetes configuration for the Shopping List API application.

## Secrets Management

This repository uses Sealed Secrets for managing sensitive information. The sealed secrets are encrypted using the public key from your cluster's sealed-secrets controller and can only be decrypted by the controller.

### Creating New Sealed Secrets

To create a new sealed secret:

1. Create a regular secret YAML file:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  namespace: my-namespace
type: Opaque
stringData:
  key1: value1
  key2: value2
```

2. Use kubeseal to encrypt it:
```bash
kubeseal --format yaml < my-secret.yaml > sealed-secret.yaml
```

3. Apply the sealed secret to your cluster:
```bash
kubectl apply -f sealed-secret.yaml
```

The sealed-secrets controller will automatically decrypt the secret and create a regular Kubernetes secret.

### Updating Secrets

To update a secret:
1. Create a new regular secret YAML with the updated values
2. Seal it using kubeseal
3. Replace the old sealed secret with the new one
4. The controller will automatically update the Kubernetes secret

### Development vs Production

The repository contains separate sealed secrets for development and production environments in their respective overlay directories. Each environment's secrets are encrypted specifically for that environment's namespace.

## Structure

```
.
├── base/                   # Base Kubernetes manifests
│   ├── 10-namespace.yaml           # Namespace definition
│   ├── 20-mysql-secret.yaml        # MySQL secrets
│   ├── 21-api-secret.yaml          # API secrets
│   ├── 30-mysql-pvc.yaml          # MySQL persistent volume claim
│   ├── 40-mysql-deployment.yaml    # MySQL deployment
│   ├── 41-mysql-service.yaml       # MySQL service
│   ├── 50-api-deployment.yaml      # API deployment
│   ├── 51-api-service.yaml         # API service
│   ├── 60-api-ingress.yaml        # API ingress
│   └── kustomization.yaml
└── overlays/              # Environment-specific configurations
    ├── dev/              # Development environment
    │   ├── kustomization.yaml
    │   ├── 20-mysql-secret.yaml
    │   └── 21-api-secret.yaml
    └── prod/             # Production environment
        ├── kustomization.yaml
        ├── 20-mysql-secret.yaml
        ├── 21-api-secret.yaml
        └── 50-api-deployment.yaml
```

## File Ordering

The files are prefixed with numbers to indicate their deployment order:

- `10-` - Namespace definitions
- `20-` - Secrets and configuration
- `30-` - Storage resources
- `40-` - Database components
- `50-` - Application components
- `60-` - Network components

This ordering ensures that dependencies are created in the correct order (e.g., namespace before resources, secrets before deployments that need them).

## Usage

### Prerequisites

- Kubernetes cluster
- kubectl installed and configured
- kustomize installed (or kubectl version >= 1.14)

### Deploying to Development

```bash
kubectl apply -k overlays/dev
```

### Deploying to Production

```bash
kubectl apply -k overlays/prod
```

## Configuration

### Secrets

The repository contains example secrets for demonstration. In a real environment:

1. Never commit actual secrets to the repository
2. Use a secure secret management solution (e.g., HashiCorp Vault, Sealed Secrets)
3. Update the secrets according to your environment

### Image Registry

Update the image registry in the overlay kustomization files:

```yaml
images:
  - name: shopping-list-api
    newName: your-registry/shopping-list-api
    newTag: dev  # or prod
```

## Monitoring

The application exposes a `/health` endpoint for kubernetes probes.

## Resources

- Development environment uses minimal resources
- Production environment has:
  - Higher replica count (3)
  - More CPU and memory resources
  - Additional monitoring and logging (to be configured)
