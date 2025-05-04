# SiYuan on Kubernetes

This repository contains Kubernetes configuration files for deploying SiYuan note-taking application on Kubernetes.

## Overview

This setup deploys the [siyuan-unlock](https://github.com/appdev/siyuan-unlock) version of SiYuan, which allows synchronization features without requiring account login.

## Components

- **ConfigMap**: Environment variables for SiYuan configuration
- **Secret**: Secure storage for sensitive information like access passwords
- **PersistentVolumeClaim**: For storing SiYuan workspaces and data
- **Deployment**: The SiYuan application container configuration
- **Service**: Internal networking configuration
- **Ingress**: External access configuration

## Prerequisites

- Kubernetes cluster (minikube, K3s, EKS, GKE, AKS, etc.)
- kubectl command-line tool
- Optional: Ingress controller (like nginx-ingress) for external access

## Installation

1. Clone this repository:
   ```bash
   git clone siyuan-unlock-k8s
   cd siyuan
   ```

2. Update the secret value:
   Edit `kubernetes/secret.yaml` and replace `your-secure-access-code` with your preferred access password.

3. Apply the Kubernetes manifests:
   ```bash
   kubectl apply -f kubernetes/secret.yaml
   kubectl apply -f kubernetes/configmap.yaml
   kubectl apply -f kubernetes/pvc.yaml
   kubectl apply -f kubernetes/deployment.yaml
   kubectl apply -f kubernetes/service.yaml
   kubectl apply -f kubernetes/ingress.yaml  # if using Ingress
   ```

## Configuration

### Environment Variables

Environment variables are managed in two locations:

1. **ConfigMap** (`kubernetes/configmap.yaml`):
   - `LANG` and `LC_ALL`: Language settings (default: en_US.UTF-8)
   - `UI_THEME`: UI theme setting (default: dark)
   - `APPEARANCE_MODE`: Appearance mode setting (default: auto)

2. **Secret** (`kubernetes/secret.yaml`):
   - `ACCESSAUTHCODE`: Secure access password

### Access Options

1. **Port Forwarding**:
   ```bash
   kubectl port-forward svc/siyuan 6806:6806
   ```
   Then visit `http://localhost:6806` in your browser.

2. **Ingress**: Configure your DNS to point `siyuan.example.com` to your Ingress controller.

## Security Best Practices

- **Never commit secrets with real values to Git**. The provided secret.yaml contains a placeholder.
- Consider using a secure secret management solution like Hashicorp Vault or AWS Secrets Manager for production.
- You can also use Sealed Secrets or External Secrets for GitOps workflows.
- For local development, you can use a .env file (ignored by Git) to store the real values, then create the secret using: 
  ```bash
  kubectl create secret generic siyuan-secret --from-env-file=.env
  ```

## Customization

- **Resource Limits**: For production use, consider adding resource limits to the Deployment.
- **Persistent Storage**: Adjust the PVC size as needed for your data requirements.
- **Security**: Consider implementing network policies for additional security.

## Troubleshooting

- Check pod status: `kubectl get pods -l app=siyuan`
- View logs: `kubectl logs -l app=siyuan`
- Describe pod: `kubectl describe pod -l app=siyuan`

## License

This Kubernetes configuration is provided under the MIT License. SiYuan itself has its own licensing terms.