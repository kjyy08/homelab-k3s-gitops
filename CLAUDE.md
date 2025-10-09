# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository manages a homelab K3s cluster infrastructure using GitOps principles with Kustomize. The infrastructure is organized into core components and applications, deployed across multiple namespaces.

## Architecture

### Structure
The repository follows a layered Kustomize structure:
- **Root kustomization.yaml**: Orchestrates all core and application deployments
- **core/**: Infrastructure components (storage, ingress, cert-manager)
- **applications/**: Service deployments organized by namespace

### Namespaces
- `observability`: Monitoring stack (Grafana, Prometheus, Loki, Tempo, OpenTelemetry Collector, Node Exporter, Kube State Metrics)
- `automation`: Workflow automation (n8n with PostgreSQL backend)
- `wiki`: Knowledge management (BookStack with MariaDB)
- `kube-system`: Core infrastructure (cert-manager)

### Key Patterns
1. Each application directory contains:
   - `namespace.yaml`: Namespace definition
   - `kustomization.yaml`: Resource aggregation
   - Application subdirectories with deployment manifests and ConfigMaps

2. Secrets are managed separately and NOT committed to git - manifests reference them via secretKeyRef

3. StatefulSets are used for stateful applications (Grafana, n8n, databases) with volumeClaimTemplates

4. Storage classes referenced:
   - `bulk-hdd`: General persistent storage
   - `local-path`: K3s default local path storage

## Common Commands

### Deployment
```bash
# Deploy entire infrastructure
kubectl apply -k .

# Deploy specific layer
kubectl apply -k core/
kubectl apply -k applications/observability/
kubectl apply -k applications/automation/
kubectl apply -k applications/wiki/

# Preview changes before applying
kubectl diff -k .
```

### Debugging and Inspection
```bash
# Check all pods across namespaces
kubectl get pods --all-namespaces

# Check specific namespace
kubectl get pods -n observability
kubectl get pods -n automation
kubectl get pods -n wiki

# View logs
kubectl logs -n observability deployment/grafana
kubectl logs -n automation statefulset/n8n
kubectl logs -n wiki deployment/bookstack

# Describe resources
kubectl describe pod -n observability <pod-name>
kubectl get pvc --all-namespaces

# Check Ingress and cert-manager
kubectl get ingress --all-namespaces
kubectl get certificate --all-namespaces
kubectl get clusterissuer
```

### Validation
```bash
# Validate Kustomize structure
kustomize build . > /dev/null

# Validate specific overlay
kustomize build applications/observability/ > /dev/null

# Render manifests without applying
kubectl kustomize .
```

## Creating Secrets

Before deploying applications, required secrets must be created:

```bash
# Grafana
kubectl create secret generic grafana-secret -n observability \
  --from-literal=GF_SECURITY_ADMIN_USER=admin \
  --from-literal=GF_SECURITY_ADMIN_PASSWORD=<password>

# n8n
kubectl create secret generic n8n-secret -n automation \
  --from-literal=N8N_BASIC_AUTH_USER=<user> \
  --from-literal=N8N_BASIC_AUTH_PASSWORD=<password> \
  --from-literal=N8N_ENCRYPTION_KEY=<encryption-key>

# BookStack (if using secrets)
kubectl create secret generic bookstack-secret -n wiki \
  --from-literal=APP_KEY=<app-key>
```

## Adding New Applications

1. Create application directory under `applications/<namespace>/<app-name>/`
2. Add manifests (deployment, service, configmap)
3. Update `applications/<namespace>/kustomization.yaml` to include new resources
4. If new namespace: create `namespace.yaml` and add to root `kustomization.yaml`
5. For external access: add Ingress resource in `core/ingress/` and update its kustomization
6. Apply with `kubectl apply -k applications/<namespace>/`

## Modifying Existing Applications

1. Edit YAML manifests directly in application subdirectories
2. Validate changes: `kubectl diff -k applications/<namespace>/`
3. Apply: `kubectl apply -k applications/<namespace>/`
4. Monitor rollout: `kubectl rollout status -n <namespace> deployment/<app-name>`

## Storage Configuration

Storage classes are defined in `core/storage/storage-classes.yaml`. Applications request storage via PVCs with `storageClassName` specified. The cluster uses local-path-provisioner as the default, with `bulk-hdd` for larger persistent storage needs.
