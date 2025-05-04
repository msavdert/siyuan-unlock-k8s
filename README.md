# SiYuan on Kubernetes

This repository contains Kubernetes configuration files for deploying SiYuan note-taking application on Kubernetes.

## Overview

This setup deploys the [siyuan-unlock](https://github.com/appdev/siyuan-unlock) version of SiYuan, which allows synchronization features without requiring account login.

## Components

- **Namespace**: Dedicated "siyuan" namespace for all resources
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
   kubectl apply -f kubernetes/namespace.yaml
   ```

4. Create ConfigMap and Secret:

   **Option 1:** Create from .env file:
   ```bash
   # Create ConfigMap from .env in the siyuan namespace
   kubectl create configmap siyuan-config --from-env-file=.env -n siyuan

   # Create Secret from .env in the siyuan namespace
   kubectl create secret generic siyuan-secret --from-env-file=.env -n siyuan
   ```

   **Option 2:** Create with auto-generated access token:
   ```bash
   # First create ConfigMap from .env
   kubectl create configmap siyuan-config --from-env-file=.env -n siyuan
   
   # Generate a random secure token and create the Secret
   RANDOM_TOKEN=$(openssl rand -base64 32)
   kubectl create secret generic siyuan-secret --from-literal=ACCESSAUTHCODE=$RANDOM_TOKEN -n siyuan
   
   # Save the token for your reference (optional)
   echo "Your generated access token is: $RANDOM_TOKEN"
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
- `ACCESSAUTHCODE`: Secure access password for SiYuan (can be auto-generated, see installation options)

### Auto-generated Access Token

When using the auto-generated token option, the deployment will use this randomly generated token as the access password for SiYuan. Make sure to note down the token when it's displayed after running the command, as you'll need it to log in to the application.

If you forget your token, you can retrieve it using:
```bash
kubectl get secret siyuan-secret -n siyuan -o jsonpath='{.data.ACCESSAUTHCODE}' | base64 --decode
```

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
- When deploying to production environments, consider using a secure secret management solution.
- Consider using the auto-generated token option for better security.
- The deployment is configured with `optional: true` for the ACCESSAUTHCODE, which means the application will start even if no access code is provided.

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