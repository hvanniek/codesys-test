# CODESYS Kubernetes Deployment

A comprehensive Kubernetes deployment solution for CODESYS virtual PLCs (Programmable Logic Controllers) with gateway support, using Helm charts and ArgoCD for GitOps-based deployment.

## Overview

This repository contains a complete infrastructure-as-code solution for deploying CODESYS virtual PLCs and gateways in Kubernetes environments. It includes:

- **CODESYS Virtual PLC**: Containerized CODESYS runtime with persistent storage
- **CODESYS Gateway**: API gateway for PLC communication and management
- **Kubernetes Integration**: Full support for persistent volumes, service accounts, and network policies
- **GitOps Ready**: ArgoCD application configuration for continuous deployment

## Project Structure

```
codesys-test/
├── files/                           # CODESYS application and configuration files
│   ├── Application.app             # CODESYS application package
│   ├── Application.crc             # Application CRC checksum
│   ├── Generator.app               # Code generator application
│   ├── Generator.crc               # Generator CRC checksum
│   ├── argo.yaml                   # ArgoCD application definition
│   ├── GroupDatabase.csv           # User group database
│   ├── UserDatabase.csv            # User account database
│   └── UserMgmtRightsDB.csv        # User management rights database
├── gateway/                         # Gateway Helm chart
│   ├── Chart.yaml                  # Helm chart metadata
│   ├── values.yaml                 # Default configuration values
│   └── templates/
│       ├── deployment.yaml         # Gateway deployment template
│       ├── service.yaml            # Gateway service template
│       └── storage.yaml            # Persistent storage configuration
├── vplc/                           # CODESYS PLC Helm chart
│   ├── Chart.yaml                  # Helm chart metadata
│   ├── values.yaml                 # Default configuration values
│   └── templates/
│       ├── namespace.yaml          # Kubernetes namespace definition
│       ├── deployment.yaml         # PLC deployment template
│       ├── service.yaml            # PLC service template
│       ├── configmap.yaml          # Configuration management
│       ├── role.yaml               # RBAC role definition
│       ├── rolebinding.yaml        # RBAC role binding
│       ├── sa.yaml                 # Service account definition
│       └── storage.yaml            # Persistent storage configuration
└── README.md                        # This file
```

## Components

### Virtual PLC (vplc/)

The virtual PLC Helm chart deploys containerized CODESYS runtime instances with:

- **Stateful Deployments**: Each PLC runs as a separate Kubernetes deployment
- **Persistent Storage**: Data and configuration volumes for runtime persistence
- **Service Accounts**: Dedicated service accounts with RBAC controls
- **Network Services**: Kubernetes services for PLC communication
- **ConfigMaps**: Dynamic configuration management

**Key Features:**
- Hostname configuration for network identification
- Multiple PLC instances support (via values)
- Persistent data and configuration storage
- Automatic namespace creation

### Gateway (gateway/)

The gateway Helm chart provides API and communication gateway functionality:

- **Deployment**: Single replica gateway service
- **Storage**: Data and configuration persistent volumes
- **Service Exposure**: Network service for external access
- **Container Image**: `quay.io/rh-ee-hvanniek/codesysgw:4.14.0.0`

**Key Features:**
- Multi-gateway support through Helm values
- Service account based RBAC
- Persistent volume claims for data retention

### Files (files/)

Application files and databases:

- **Application Packages**: CODESYS application bundles (.app, .crc)
- **User Management**: CSV databases for users and groups
- **Generator**: Code generation tool for application development
- **ArgoCD Config**: GitOps deployment specification

## Prerequisites

- Kubernetes 1.20+ cluster
- Helm 3.0+
- ArgoCD (for GitOps deployment, optional)
- Persistent volume provisioner
- Service account with cluster admin privileges

## Installation

### Using Helm Directly

1. **Deploy the vPLC:**
   ```bash
   helm install codesys-plc ./vplc -n codesys --create-namespace
   ```

2. **Deploy the Gateway:**
   ```bash
   helm install codesys-gw ./gateway -n codesys
   ```

### Using ArgoCD (GitOps)

1. Create the ArgoCD application:
   ```bash
   kubectl apply -f files/argo.yaml
   ```

2. Monitor deployment:
   ```bash
   argocd app get codesys
   ```

## Configuration

### Custom Values

Override default configuration in `helm install` with custom values:

```bash
helm install codesys-plc ./vplc -n codesys \
  --values custom-values.yaml
```

### Key Configuration Options

- **Replicas**: Configured per deployment in values.yaml
- **Image Tags**: Update container versions in values.yaml
- **Storage**: Persistent volume sizes in values.yaml
- **Network**: Service types and ports in service templates

## Storage

Both vPLC and gateway use persistent volumes with two claim types:

- **data-storage**: Application runtime data
- **conf-storage**: Configuration files and settings

Ensure your cluster has a default storage class or specify one in values.

## Networking

- **vPLC Service**: ClusterIP service for PLC communication
- **Gateway Service**: ClusterIP/LoadBalancer service for external access
- **Hostnames**: Set via deployment templates for DNS resolution

## RBAC and Security

- **Service Accounts**: Dedicated `codesys-plc-controller-sa` for PLC operations
- **Roles**: RBAC roles defined in role.yaml templates
- **Role Bindings**: Role bindings connect service accounts to roles
- **Namespaces**: Isolated namespaces for security

## Troubleshooting

### Check Deployment Status
```bash
kubectl get deployments -n codesys
kubectl get pods -n codesys
kubectl describe pod <pod-name> -n codesys
```

### View Logs
```bash
kubectl logs <pod-name> -n codesys
```

### Verify Storage
```bash
kubectl get pvc -n codesys
kubectl get pv
```

### Check Service Connectivity
```bash
kubectl get svc -n codesys
```

## GitOps Workflow

This repository is designed for continuous deployment via ArgoCD:

1. Push changes to `main` branch
2. ArgoCD detects changes automatically
3. Deployment updates in `codesys` namespace
4. Monitor via ArgoCD dashboard or CLI

## Support and Documentation

- [CODESYS Documentation](https://www.codesys.com/en/)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)


