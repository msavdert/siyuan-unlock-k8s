# SiYuan on Kubernetes

This repository contains Kubernetes configuration files for deploying SiYuan note-taking application on Kubernetes.

## Overview

This setup deploys the [siyuan-unlock](https://github.com/appdev/siyuan-unlock) version of SiYuan, which allows synchronization features without requiring account login.

## Components

- **ConfigMap**: Environment variables for SiYuan configuration
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

2. Apply the Kubernetes manifests:
   ```bash
   kubectl apply -f kubernetes/configmap.yaml
   kubectl apply -f kubernetes/pvc.yaml
   kubectl apply -f kubernetes/deployment.yaml
   kubectl apply -f kubernetes/service.yaml
   kubectl apply -f kubernetes/ingress.yaml  # if using Ingress
   ```

## Configuration

### Environment Variables

All environment variables are managed in the `kubernetes/configmap.yaml` file. Key settings include:

- `LANG` and `LC_ALL`: Language settings (default: en_US.UTF-8)
- `ACCESSAUTHCODE`: Optional access password (commented out by default)
- `UI_THEME`: UI theme setting (default: dark)
- `APPEARANCE_MODE`: Appearance mode setting (default: auto)

### Access Options

1. **Port Forwarding**:
   ```bash
   kubectl port-forward svc/siyuan 6806:6806
   ```
   Then visit `http://localhost:6806` in your browser.

2. **Ingress**: Configure your DNS to point `siyuan.example.com` to your Ingress controller.

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