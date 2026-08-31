# Vprofile Helm Chart

A production-ready Helm chart for deploying the Vprofile application stack on Kubernetes with AWS EKS support.

## Overview

This Helm chart deploys a complete microservices application stack consisting of:
- **Vprofile App** - Java/Spring Boot application
- **MySQL Database** - Persistent database backend
- **Memcached** - In-memory caching layer
- **RabbitMQ** - Message queue broker

## Features

- ✅ AWS ALB (Application Load Balancer) ingress support
- ✅ AWS EBS gp2 storage class for database persistence
- ✅ SSL/TLS termination with AWS ACM certificates
- ✅ Conditional ingress and Docker registry secret rendering
- ✅ Init containers for dependency ordering
- ✅ Secret management for database and message queue credentials
- ✅ Configurable resource replicas and container ports
- ✅ Docker registry authentication support

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- AWS EKS cluster (for production use)
- AWS ALB Ingress Controller installed
- AWS ACM certificate (for SSL/TLS)

## Installation

### Basic Installation

```bash
# Add the chart repository (if applicable)
helm repo add vprofile https://example.com/charts

# Install the chart with default values
helm install vprofile ./helm/vprofile

# Or with a custom release name
helm install my-vprofile ./helm/vprofile
```

### Installation with Custom Values

```bash
# Install with custom values file
helm install vprofile ./helm/vprofile -f custom-values.yaml

# Or override specific values
helm install vprofile ./helm/vprofile \
  --set ingress.host=myapp.example.com \
  --set dockerregistry.enabled=true \
  --set dockerregistry.server=docker.io \
  --set dockerregistry.username=myuser \
  --set dockerregistry.password=mypass \
  --set dockerregistry.email=myemail@example.com
```

## Configuration

### Application Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `app.name` | App deployment name | `vproapp` |
| `app.image` | App container image | `vprocontainers/vprofileapp` |
| `app.tag` | App image tag | `latest` |
| `app.replicas` | Number of app replicas | `1` |
| `app.containerPort` | Container port | `8080` |
| `app.servicePort` | Service port | `8080` |

### Database Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `db.name` | Database deployment name | `vprodb` |
| `db.image` | Database image | `vprocontainers/vprofiledb` |
| `db.tag` | Database image tag | `latest` |
| `db.replicas` | Number of DB replicas | `1` |
| `db.containerPort` | Database port | `3306` |
| `db.servicePort` | Service port | `3306` |
| `db.storageClass` | Storage class for EBS | `gp2` |
| `db.storageSize` | Database volume size | `3Gi` |
| `db.defaultUser` | Default database user | `root` |

### Memcached Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `memcached.name` | Memcached deployment name | `vpromc` |
| `memcached.image` | Memcached image | `memcached` |
| `memcached.tag` | Memcached image tag | `latest` |
| `memcached.replicas` | Number of Memcached replicas | `1` |
| `memcached.containerPort` | Container port | `11211` |
| `memcached.servicePort` | Service port | `11211` |

### RabbitMQ Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `rabbitmq.name` | RabbitMQ deployment name | `vpromq01` |
| `rabbitmq.image` | RabbitMQ image | `rabbitmq` |
| `rabbitmq.tag` | RabbitMQ image tag | `latest` |
| `rabbitmq.replicas` | Number of RabbitMQ replicas | `1` |
| `rabbitmq.containerPort` | Container port | `5672` |
| `rabbitmq.servicePort` | Service port | `5672` |
| `rabbitmq.defaultUser` | Default RabbitMQ user | `guest` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.host` | Ingress hostname | `vprofile.salmancloud.xyz` |
| `ingress.servicePort` | Service port for ingress | `8080` |

**Ingress Annotations:**
- `kubernetes.io/ingress.class: alb` - Uses AWS ALB Controller
- `alb.ingress.kubernetes.io/scheme: internet-facing` - Internet-facing ALB
- `alb.ingress.kubernetes.io/target-type: ip` - IP target type
- `alb.ingress.kubernetes.io/certificate-arn` - AWS ACM certificate for SSL/TLS
- `alb.ingress.kubernetes.io/ssl-redirect: '443'` - Redirect HTTP to HTTPS

### Secrets Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `secrets.appSecret.name` | Secret resource name | `app-secret` |
| `secrets.appSecret.dbPassword` | Base64-encoded DB password | `dnByb2RicGFzcw==` |
| `secrets.appSecret.rmqPassword` | Base64-encoded RabbitMQ password | `Z3Vlc3Q=` |

### Docker Registry Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `dockerregistry.enabled` | Enable docker registry secret | `false` |
| `dockerregistry.server` | Docker registry server | `docker.io` |
| `dockerregistry.username` | Registry username | `` |
| `dockerregistry.password` | Registry password | `` |
| `dockerregistry.email` | Registry email | `` |

## Usage Examples

### Deploy with Docker Registry Authentication

```bash
helm install vprofile ./helm/vprofile \
  --set dockerregistry.enabled=true \
  --set dockerregistry.server=gcr.io \
  --set dockerregistry.username=_json_key \
  --set dockerregistry.password="$(cat ~/key.json)" \
  --set dockerregistry.email=user@example.com
```

### Update Database Storage Size

```bash
helm upgrade vprofile ./helm/vprofile \
  --set db.storageSize=10Gi
```

### Scale Applications

```bash
helm upgrade vprofile ./helm/vprofile \
  --set app.replicas=3 \
  --set db.replicas=1
```

### Disable Ingress

```bash
helm install vprofile ./helm/vprofile \
  --set ingress.enabled=false
```

## Deployment Structure

### Templates

| File | Description |
|------|-------------|
| `app-deployment.yaml` | Vprofile application deployment with init containers |
| `db-deployment.yaml` | MySQL database deployment with persistent volume |
| `mc-deployment.yaml` | Memcached deployment |
| `rmq-deployment.yaml` | RabbitMQ deployment |
| `services.yaml` | Kubernetes services for all components |
| `ingress.yaml` | AWS ALB ingress (conditional) |
| `secret.yaml` | Application secrets |
| `pvc.yaml` | PersistentVolumeClaim for database |
| `dockerregistry-secret.yaml` | Docker registry secret (conditional) |

### Init Containers

The application deployment includes init containers that wait for database and memcached services to be available:

```yaml
- init-mydb: Waits for vprodb service DNS resolution
- init-memcache: Waits for vprocache01 service DNS resolution
```

### Persistent Storage

The database deployment uses a PersistentVolumeClaim with:
- **Storage Class**: `gp2` (AWS EBS General Purpose)
- **Access Mode**: ReadWriteOnce
- **Size**: Configurable (default: 3Gi)

## Verification

### Check Deployment Status

```bash
# List all deployments
kubectl get deployments

# Check pod status
kubectl get pods

# View services
kubectl get services

# Verify ingress
kubectl get ingress
```

### View Logs

```bash
# App logs
kubectl logs -l app=vproapp

# Database logs
kubectl logs -l app=vprodb

# Memcached logs
kubectl logs -l app=vpromc

# RabbitMQ logs
kubectl logs -l app=vpromq01
```

### Port Forwarding (for local testing)

```bash
# Forward app port
kubectl port-forward svc/vproapp-service 8080:8080

# Forward database port
kubectl port-forward svc/vprodb 3306:3306

# Forward memcached port
kubectl port-forward svc/vprocache01 11211:11211

# Forward RabbitMQ port
kubectl port-forward svc/vpromq01 5672:5672
```

## Troubleshooting

### Init Containers Not Completing

If init containers are stuck waiting for services:

```bash
# Check which services are available
kubectl get services

# Verify DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
# Inside pod: nslookup vprodb
```

### Database Connection Issues

```bash
# Test MySQL connectivity
kubectl run -it --rm mysql-client --image=mysql:latest --restart=Never -- \
  mysql -h vprodb -u root -p
```

### Storage Issues

```bash
# Check PersistentVolumeClaim status
kubectl get pvc

# Check PersistentVolume status
kubectl get pv

# Describe PVC for events
kubectl describe pvc db-pv-claim
```

## Upgrading

### Upgrade Chart

```bash
# Update values
helm upgrade vprofile ./helm/vprofile -f values.yaml

# With new values
helm upgrade vprofile ./helm/vprofile \
  --set app.tag=v2.0 \
  --set db.storageSize=5Gi
```

### Rollback

```bash
# List releases
helm history vprofile

# Rollback to previous release
helm rollback vprofile

# Rollback to specific revision
helm rollback vprofile 2
```

## Uninstallation

```bash
# Delete the release
helm uninstall vprofile
```

**Note**: PersistentVolumes and associated data are retained after chart deletion for data safety. To remove them manually:

```bash
kubectl delete pvc db-pv-claim
kubectl delete pv <pv-name>
```

## Security Considerations

1. **Secrets Management**: Update default passwords in `values.yaml`
2. **Image Pull Secrets**: Enable `dockerregistry.enabled` for private registries
3. **Resource Limits**: Add CPU/memory limits in production
4. **Network Policies**: Implement network policies for inter-service communication
5. **RBAC**: Configure appropriate ServiceAccounts and roles

## AWS EKS Specifics

### ALB Ingress Controller Setup

```bash
# Install ALB ingress controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system --set clusterName=my-cluster
```

### Storage Class Configuration

The chart uses `gp2` storage class which is standard on AWS EKS:

```bash
# Verify storage class
kubectl get storageclass

# If needed, create gp2 storage class:
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
  iops: "3000"
  throughput: "125"
EOF
```

## Support and Contribution

For issues, questions, or contributions, please contact the development team or open an issue in the repository.

## License

This Helm chart is provided as-is for internal use within the organization.
