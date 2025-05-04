# SiYuan on Kubernetes

This repository contains Kubernetes configuration files for deploying SiYuan note-taking application on Kubernetes.

## Overview

This setup deploys the [siyuan-unlock](https://github.com/appdev/siyuan-unlock) version of SiYuan, which allows synchronization features without requiring account login.

## Components

- **ConfigMap**: Environment variables for SiYuan configuration
- **Secret**: Secure storage for sensitive information like access passwords
- **PersistentVolumeClaim**: For storing SiYuan workspaces and data
- **Deployment**: The SiYuan application container configuration
- **Service**: External access configuration (LoadBalancer, NodePort, or ClusterIP)
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
   kubectl create secret generic siyuan-secret --from-literal=SIYUAN_ACCESS_AUTH_CODE=$RANDOM_TOKEN -n siyuan
   
   # Save the token for your reference (optional)
   echo "Your generated access token is: $RANDOM_TOKEN"
   ```

5. **Service Type Configuration**:
   The default service is configured to use LoadBalancer. If you want to use another service type:
   
   - Edit the service.yaml file:
     ```bash
     # Open the service.yaml file in your editor
     nano kubernetes/service.yaml
     ```
   
   - Change the service type to your preferred type (ClusterIP or NodePort)
   
   - Or apply a modified service configuration directly:
     ```bash
     # For ClusterIP (internal access only)
     kubectl apply -f - <<EOF
     apiVersion: v1
     kind: Service
     metadata:
       name: siyuan
       namespace: siyuan
     spec:
       selector:
         app: siyuan
       ports:
       - port: 6806
         targetPort: 6806
       type: ClusterIP
     EOF
     
     # For NodePort (access via node IP and allocated port)
     kubectl apply -f - <<EOF
     apiVersion: v1
     kind: Service
     metadata:
       name: siyuan
       namespace: siyuan
     spec:
       selector:
         app: siyuan
       ports:
       - port: 6806
         targetPort: 6806
         nodePort: 30806  # You can specify a port between 30000-32767 or leave it for automatic allocation
       type: NodePort
     EOF
     ```

6. Apply the remaining Kubernetes manifests:
   ```bash
   kubectl apply -f kubernetes/pvc.yaml
   kubectl apply -f kubernetes/deployment.yaml
   kubectl apply -f kubernetes/service.yaml  # If you edited this file
   kubectl apply -f kubernetes/ingress.yaml  # if using Ingress
   ```

## Configuration

### Environment Variables

All configuration is managed in the `.env` file (copied from `.env.example`):

- `LANG` and `LC_ALL`: Language settings (default: en_US.UTF-8)
- `UI_THEME`: UI theme setting (default: dark)
- `APPEARANCE_MODE`: Appearance mode setting (default: auto)
- `SIYUAN_ACCESS_AUTH_CODE`: Secure access password for SiYuan (can be auto-generated, see installation options)

### Auto-generated Access Token

When using the auto-generated token option, the deployment will use this randomly generated token as the access password for SiYuan. Make sure to note down the token when it's displayed after running the command, as you'll need it to log in to the application.

If you forget your token, you can retrieve it using:
```bash
kubectl get secret siyuan-secret -n siyuan -o jsonpath='{.data.SIYUAN_ACCESS_AUTH_CODE}' | base64 --decode
```

### Access Options

#### 1. LoadBalancer (default)
The service is configured as `LoadBalancer` type by default, which will provision an external IP address through your cloud provider.

```bash
# Get the external IP
kubectl get svc siyuan -n siyuan

# Access URL
http://<EXTERNAL-IP>:6806
```

**Pros**: 
- Simple direct access from anywhere
- Automatic load balancing
- Fixed external IP

**Cons**: 
- Costs money in most cloud providers
- Not available in some environments (like local Minikube)

#### 2. NodePort
With NodePort, the service is exposed on each node's IP at a static port.

```bash
# Change the service type
kubectl patch svc siyuan -n siyuan -p '{"spec": {"type": "NodePort"}}'

# Get the allocated NodePort
kubectl get svc siyuan -n siyuan

# Access URL
http://<NODE-IP>:<NODE-PORT>
```

**Pros**:
- Works in any Kubernetes cluster
- No need for an Ingress controller
- Free (no additional cloud resources)

**Cons**: 
- Port range limited to 30000-32767
- Node IP might change
- Requires external access to the nodes

#### 3. ClusterIP with Port Forwarding
With ClusterIP, the service is only accessible within the cluster, but you can use port-forwarding for temporary access:

```bash
# Change the service type
kubectl patch svc siyuan -n siyuan -p '{"spec": {"type": "ClusterIP"}}'

# Port forwarding
kubectl port-forward svc/siyuan 6806:6806 -n siyuan

# Access URL (while port forwarding is active)
http://localhost:6806
```

**Pros**:
- Most secure option
- Works in any environment
- Free (no additional cloud resources)

**Cons**:
- Requires keeping the port-forwarding command running
- Only accessible on the machine running the command
- Temporary access

#### 4. Ingress
For more advanced routing, using an Ingress resource with a domain name:

```bash
# Make sure you have an Ingress controller installed
# Edit kubernetes/ingress.yaml to set your domain name
kubectl apply -f kubernetes/ingress.yaml -n siyuan

# Access URL
http://siyuan.your-domain.com
```

**Pros**:
- Nice URL with your own domain
- Can use SSL/TLS for HTTPS
- Can host multiple applications with different paths
- Single external IP for multiple services

**Cons**:
- Requires an Ingress controller
- Requires DNS configuration
- More complex setup

## File Management

- **`.env.example`**: Template file containing placeholder values, committed to Git
- **`.env`**: Your actual configuration file with real values, excluded from Git
- **`kubernetes/*.yaml`**: Kubernetes configuration files

## Security Best Practices

- **Keep your `.env` file private**: This file contains sensitive information and should not be committed to Git.
- When deploying to production environments, consider using a secure secret management solution.
- Consider using the auto-generated token option for better security.
- The deployment is configured with `optional: true` for the SIYUAN_ACCESS_AUTH_CODE, which means the application will start even if no access code is provided.
- For production, consider setting up network policies to restrict traffic to the SiYuan pod.
- If using Ingress, consider adding TLS configuration for secure HTTPS access.

## Customization

- **Resource Limits**: For production use, consider adding resource limits to the Deployment:
  ```yaml
  resources:
    limits:
      cpu: "1"
      memory: "1Gi"
    requests:
      cpu: "200m"
      memory: "256Mi"
  ```

- **Persistent Storage**: Adjust the PVC size in `kubernetes/pvc.yaml` as needed for your data requirements.

- **High Availability**: For critical deployments, consider increasing replicas in the deployment:
  ```yaml
  replicas: 2  # Or more as needed
  ```

## Troubleshooting

- Check pod status: `kubectl get pods -l app=siyuan -n siyuan`
- View logs: `kubectl logs -l app=siyuan -n siyuan`
- Describe pod: `kubectl describe pod -l app=siyuan -n siyuan`
- Check service status: `kubectl get svc siyuan -n siyuan`
- Check ingress status: `kubectl describe ingress siyuan-ingress -n siyuan`
- Check persistent volume: `kubectl get pvc siyuan-data -n siyuan`
- Check node port: `kubectl get svc siyuan -n siyuan -o=jsonpath='{.spec.ports[0].nodePort}'`

### Common Issues and Solutions

1. **Pod fails to start**:
   - Check logs: `kubectl logs -l app=siyuan -n siyuan`
   - Verify secret exists: `kubectl get secret siyuan-secret -n siyuan`

2. **Cannot access SiYuan**:
   - For LoadBalancer: Ensure external IP is assigned
   - For NodePort: Make sure node is accessible and firewall allows the port
   - For Ingress: Verify Ingress controller is running and DNS is configured

3. **Data persistence issues**:
   - Check PVC status: `kubectl get pvc siyuan-data -n siyuan`
   - Verify volume mounts: `kubectl describe pod -l app=siyuan -n siyuan`

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