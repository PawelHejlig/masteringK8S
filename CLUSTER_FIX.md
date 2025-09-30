# Kubernetes Cluster Authentication Fix

## Problem
The kubeconfig file had invalid configuration preventing cluster access.

## Errors Encountered
- `Please enter Username:` - Missing authentication credentials
- `dial tcp: lookup <HOST_IP>: no such host` - Invalid server address with placeholder
- `x509: certificate signed by unknown authority` - TLS verification issue with self-signed certificates
- `invalid bearer token` - Incorrect authentication token

## Environment
- Kubebuilder test environment
- API Server: `10.0.3.212:6443`
- Authentication: Token-based (`/tmp/token.csv`)
- Authorization: `AlwaysAllow` mode

## Solution Steps

### 1. Fixed Server IP Address
Replaced the `<HOST_IP>` placeholder with actual IP:
```bash
sed -i 's|<HOST_IP>|10.0.3.212|g' ~/.kube/config
```

### 2. Disabled TLS Verification
Since the cluster uses self-signed certificates:
```bash
kubectl config set-cluster local --insecure-skip-tls-verify=true
```

### 3. Fixed File Permissions
```bash
sudo chown $(id -u):$(id -g) ~/.kube/config
chmod 600 ~/.kube/config
```

### 4. Added Authentication Token
Used the token from `/tmp/token.csv`:
```bash
kubectl config set-credentials admin --token=1234567890
```

## Final Working Configuration
```yaml
apiVersion: v1
clusters:
- cluster:
    insecure-skip-tls-verify: true
    server: https://10.0.3.212:6443
  name: local
contexts:
- context:
    cluster: local
    user: admin
  name: local
current-context: local
kind: Config
users:
- name: admin
  user:
    token: 1234567890
```

## Verification
```bash
$ kubectl get nodes
NAME                STATUS   ROLES    AGE   VERSION
codespaces-d37aef   Ready    master   8h    v1.30.0
```

## Root Cause
The kubeconfig was generated with a template placeholder `<HOST_IP>` that wasn't replaced with the actual IP address, and the user credentials section was empty. This is common in kubebuilder test environments where configuration needs to be customized per deployment.
