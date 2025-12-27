# platform-bootstrap

End-to-end platform bootstrap and CI/CD setup using Ansible, Docker, Kubernetes, Helm, and Jenkins.

## Purpose

This repository demonstrates how I bootstrapped a fresh Ubuntu server using Ansible
and deployed a containerised Flask application on Kubernetes using Helm and Jenkins.

## Part 1: Infrastructure Bootstrap

- Bootstrapped a headless Ubuntu server using Ansible
- Updated system packages
- Disabled swap for Kubernetes
- Configured UFW firewall with minimal required ports
- Installed Docker and enabled non-root access
- Configured a non-root user with:
  - SSH key-based access
  - Passwordless sudo privileges
- Deployed a test NGINX container to validate Docker setup

## Part 2: Docker — Containerization

- Built a lightweight Flask-based web application
- Implemented a `/health` endpoint for container health checks
- Created a multi-stage Dockerfile to optimise image size
- Added `.dockerignore` to reduce build context
- Successfully built and ran the container locally

## Part 3: Kubernetes + Helm Deployment

The containerised Flask application is deployed to a single-node Kubernetes cluster (MicroK8s) using Helm.

This demonstrates core Kubernetes deployment patterns, service exposure, ingress routing, configuration management, and autoscaling.

### Kubernetes Resources Used

- Deployment – Replica-based application workload
- Service (ClusterIP) – Internal service exposure
- Ingress (NGINX) – External HTTP access
- ConfigMap – Application configuration
- Horizontal Pod Autoscaler (HPA) – CPU-based autoscaling
- Helm – Application packaging and lifecycle management

### Kubernetes Components

#### Deployment
- Initial replicas: 2
- Container image: `platform-bootstrap-app:latest`
- Health checks:
  - `/health` endpoint used for liveness and readiness probes
- Resource management:
  - CPU and memory requests & limits defined
- Configuration:
  - Application configuration injected via ConfigMap

#### Service
- Type: ClusterIP
- Port: 8080
- Routes traffic to application pods using Kubernetes labels

#### Ingress
- Ingress controller: NGINX
- Host: `app.local`
- Path: `/`
- Provides HTTP access to the application  
  Note: `app.local` is mapped to `127.0.0.1` in `/etc/hosts` for local testing.

#### ConfigMap
Stores non-sensitive application configuration:
- `APP_NAME`
- `APP_ENV`  
Injected into pods using `envFrom`.

#### Horizontal Pod Autoscaler (HPA)
- Minimum replicas: 2
- Maximum replicas: 5
- Metric: CPU utilisation
- Target CPU threshold: 50%

### Helm Chart Structure

```text
helm/platform-bootstrap-app/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── configmap.yaml
    └── hpa.yaml
```
### Deployment Steps

#### Configure kubeconfig for MicroK8s

```bash
microk8s config > ~/.kube/config
chmod 600 ~/.kube/config
```
#### Install the Helm release
```bash
helm install platform-bootstrap-app helm/platform-bootstrap-app
```
#### Upgrade the release after changes
```bash
helm upgrade platform-bootstrap-app helm/platform-bootstrap-app
```
### Verification Steps

#### Verify Kubernetes resources

```bash
kubectl get deployment
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl get configmap
kubectl get hpa
```
### Access the Application

Once the deployment is verified, the application can be accessed via the ingress.

```bash
curl http://app.local
curl http://app.local/health
```
### Observed Behaviour

During load testing, the following behaviour was observed:

- When CPU usage exceeded 50%:
  - Horizontal Pod Autoscaler (HPA) automatically scaled the application pods
  - Replica count increased up to a maximum of 5 pods

- After the load stopped:
  - CPU utilisation dropped below the target threshold
  - Pods gradually scaled back down to the minimum of 2 replicas

This confirms that:

- Metrics Server is operational
- HPA is correctly configured and functioning as expected
- Kubernetes scale-up and scale-down behaviour follows stabilisation rules

## Part 4: Secrets Management

Sensitive configuration is managed using Kubernetes Secrets and Helm, ensuring credentials are never hardcoded in source code, images, or ConfigMaps. 
This follows security best practices and the 12-Factor App principles.

### Secrets Used

The following sensitive values are stored as Kubernetes Secrets:

- `DB_USERNAME`
- `DB_PASSWORD`
- `APP_SECRET_KEY`

These values are templated via Helm and can be **overridden per environment**
without modifying application code or manifests.

### Helm-Based Secret Management

Secrets are created dynamically at deploy time using Helm templates:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "platform-bootstrap-app.fullname" . }}-secrets
type: Opaque
stringData:
  DB_USERNAME: {{ .Values.secrets.dbUsername | quote }}
  DB_PASSWORD: {{ .Values.secrets.dbPassword | quote }}
  APP_SECRET_KEY: {{ .Values.secrets.appSecretKey | quote }}
```

### Injecting Secrets into Pods

Secrets are injected into the application container as environment variables
using `envFrom.secretRef`.

```yaml
envFrom:
  - configMapRef:
      name: platform-bootstrap-app
  - secretRef:
      name: platform-bootstrap-app-secrets
```

### Verification Steps

#### Verify Secret Exists

```bash
kubectl get secret platform-bootstrap-app-secrets
kubectl describe secret platform-bootstrap-app-secrets
```
#### Verify Secrets Inside Pod
``` bash
kubectl exec -it <pod-name> -- env | grep -E 'DB_|APP_'
```
### Outcome

- Kubernetes Secrets are securely managed using Helm
- Secrets are injected into pods using environment variables
- Application remains fully functional after secrets integration
- Security best practices are enforced throughout the deployment lifecycle


