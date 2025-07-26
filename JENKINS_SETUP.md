# Jenkins Setup for Secure Secrets Management

## Prerequisites

1. Jenkins server with Docker and kubectl installed
2. Access to Docker registry
3. Kubernetes cluster access

## Jenkins Credentials Setup

### 1. Create Jenkins Credentials

Go to **Jenkins > Manage Jenkins > Manage Credentials > System > Global credentials** and add:

#### MongoDB Credentials
- **ID**: `mongodb-username`
- **Type**: Secret text
- **Secret**: Your MongoDB username

- **ID**: `mongodb-password`  
- **Type**: Secret text
- **Secret**: Your MongoDB password

- **ID**: `mongodb-database`
- **Type**: Secret text
- **Secret**: Your MongoDB database name

#### Docker Registry Credentials
- **ID**: `docker-registry`
- **Type**: Username with password
- **Username**: Your Docker registry username
- **Password**: Your Docker registry password

#### Kubernetes Credentials
- **ID**: `kubeconfig`
- **Type**: Secret file
- **Secret**: Your kubeconfig file content

### 2. Required Jenkins Plugins

Install these plugins:
- Credentials Plugin
- Docker Pipeline
- Kubernetes CLI
- Kubernetes Credentials

### 3. Pipeline Configuration

The pipeline will:
1. Build Docker images with secrets from Jenkins credentials
2. Push images to registry
3. Create Kubernetes secrets securely
4. Deploy to cluster

## Security Best Practices

### 1. Never Commit Secrets
- ✅ Use Jenkins credentials
- ✅ Use Kubernetes secrets
- ✅ Use environment variables
- ❌ Never hardcode passwords in files

### 2. Rotate Credentials Regularly
- Change MongoDB passwords monthly
- Rotate Docker registry tokens
- Update kubeconfig when needed

### 3. Access Control
- Limit Jenkins access to authorized users
- Use RBAC for Kubernetes access
- Audit credential usage

## Troubleshooting

### Common Issues

1. **Docker build fails**: Check Docker registry credentials
2. **Kubernetes deployment fails**: Verify kubeconfig and namespace permissions
3. **MongoDB connection fails**: Ensure secrets are created before deployment

### Debug Commands

```bash
# Check if secrets exist
kubectl get secrets -n biglab

# Verify MongoDB secret
kubectl get secret mongodb-secret -n biglab -o yaml

# Check pod logs
kubectl logs -f deployment/mongodb -n biglab
```

## Alternative: External Secrets Operator

For production, consider using External Secrets Operator:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: mongodb-secret
spec:
  secretStoreRef:
    name: mongodb-secret-store
    kind: SecretStore
  target:
    name: mongodb-secret
  data:
    - secretKey: username
      remoteRef:
        key: mongodb/username
    - secretKey: password
      remoteRef:
        key: mongodb/password
``` 