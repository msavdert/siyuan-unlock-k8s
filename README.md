# SiYuan on Kubernetes

This repository contains Kubernetes configuration files for deploying SiYuan note-taking application on Kubernetes.

## Overview

This setup deploys the [siyuan-unlock](https://github.com/appdev/siyuan-unlock) version of SiYuan, which allows synchronization features without requiring account login.

## Components

- **ConfigMap**: Environment variables for SiYuan configuration
- **Secret**: Secure storage for sensitive information like access passwords
- **PersistentVolumeClaim**: For storing SiYuan workspaces and data
- **Deployment**: The SiYuan application container configuration
- **Service**: External access via LoadBalancer
- **Ingress**: Alternative external access configuration (optional)

## Prerequisites

- Kubernetes cluster (minikube, K3s, EKS, GKE, AKS, etc.)
- kubectl command-line tool
- Optional: Ingress controller (like nginx-ingress) for Ingress-based access

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/msavdert/siyuan-unlock-k8s
   cd siyuan
   ```

2. Set up environment variables:
   ```bash
   cp .env.example .env
   ```
   Then edit the `.env` file to set your configuration values

3. Create the namespace first:
   ```bash
   kubectl create namespace siyuan
   ```

4. Create ConfigMap and Secret directly from the .env file:
   ```bash
   # Create ConfigMap from .env in the siyuan namespace
   kubectl create configmap siyuan-config --from-env-file=.env -n siyuan

   # Create Secret from .env in the siyuan namespace
   kubectl create secret generic siyuan-secret --from-env-file=.env -n siyuan
   ```

5. Apply the remaining Kubernetes manifests:
   ```bash
   kubectl apply -f kubernetes/pvc.yaml
   kubectl apply -f kubernetes/deployment.yaml
   kubectl apply -f kubernetes/service.yaml
   kubectl apply -f kubernetes/ingress.yaml  # if using Ingress
   ```

## Configuration

### Environment Variables

All configuration is managed in the `.env` file (copied from `.env.example`):

- `LANG` and `LC_ALL`: Language settings (default: en_US.UTF-8)
- `UI_THEME`: UI theme setting (default: dark)
- `APPEARANCE_MODE`: Appearance mode setting (default: auto)
- `ACCESSAUTHCODE`: Secure access password for SiYuan

### Access Options

1. **LoadBalancer** (default):
   The service is configured as `LoadBalancer` type, which will provision an external IP address through your cloud provider. You can access SiYuan at:
   ```
   http://<EXTERNAL-IP>:6806
   ```
   
   Get the external IP with:
   ```bash
   kubectl get svc siyuan -n siyuan
   ```

2. **Port Forwarding** (for local testing):
   ```bash
   kubectl port-forward svc/siyuan 6806:6806 -n siyuan
   ```
   Then visit `http://localhost:6806` in your browser.

3. **Ingress**: Configure your DNS to point `siyuan.example.com` to your Ingress controller.

## File Management

- **`.env.example`**: Template file containing placeholder values, committed to Git
- **`.env`**: Your actual configuration file with real values, excluded from Git
- **`kubernetes/*.yaml`**: Kubernetes configuration files

## Security Best Practices

- **Keep your `.env` file private**: This file contains sensitive information and should not be committed to Git.
- When deploying to production environments, consider using a secret management solution like Hashicorp Vault or cloud provider secret stores.
- You can also use Sealed Secrets or External Secrets for GitOps workflows.

## Customization

- **Resource Limits**: For production use, consider adding resource limits to the Deployment.
- **Persistent Storage**: Adjust the PVC size as needed for your data requirements.
- **Security**: Consider implementing network policies for additional security.

## Troubleshooting

- Check pod status: `kubectl get pods -l app=siyuan -n siyuan`
- View logs: `kubectl logs -l app=siyuan -n siyuan`
- Describe pod: `kubectl describe pod -l app=siyuan -n siyuan`
- Check LoadBalancer status: `kubectl get svc siyuan -n siyuan`

## Uninstalling

To completely remove SiYuan and all associated resources from your Kubernetes cluster:

1. Remove individual resources (if you want to delete specific components):
   ```bash
   kubectl delete deployment siyuan -n siyuan
   kubectl delete service siyuan -n siyuan
   kubectl delete ingress siyuan-ingress -n siyuan
   kubectl delete configmap siyuan-config -n siyuan
   kubectl delete secret siyuan-secret -n siyuan
   kubectl delete pvc siyuan-data -n siyuan
   ```

2. Or simply delete the entire namespace to remove everything at once:
   ```bash
   kubectl delete namespace siyuan
   ```
   
   Note: Deleting the namespace will remove ALL resources associated with SiYuan, including persistent volumes. This means all your data will be permanently deleted.

## License

This Kubernetes configuration is provided under the MIT License. SiYuan itself has its own licensing terms.