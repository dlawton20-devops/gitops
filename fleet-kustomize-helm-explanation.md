# Fleet, Kustomize, and Helm: How They Work Together

This document explains how Fleet integrates with Kustomize and Helm, and the differences between using Kustomize vs just Helm.

## Overview

Fleet is a GitOps tool that can work with both Kustomize and Helm, often using them together. Here's how they interact:

```
Git Repository
    ↓
Fleet (GitRepo)
    ↓
Kustomize (Processes YAML)
    ↓
Helm (Deploys Charts)
    ↓
Kubernetes Cluster
```

## How Fleet Works with Kustomize

### 1. Fleet's Processing Pipeline

Fleet processes your Git repository in this order:

1. **GitRepo** - Fleet watches your Git repository
2. **Kustomize** - Fleet runs Kustomize on your YAML files
3. **Bundle** - Fleet creates Bundles from the processed YAML
4. **BundleDeployment** - Fleet deploys to target clusters
5. **Helm** - If using Helm charts, Fleet uses Helm to install them

### 2. Kustomize in Fleet

Kustomize is a native Kubernetes configuration management tool that:
- Processes YAML files using overlays and patches
- Generates ConfigMaps and Secrets from files
- Applies transformations to resources
- Combines multiple resources into a single deployment

**Example Kustomize Structure:**
```
monitoring/
├── base/
│   ├── kustomization.yaml
│   ├── helmrelease.yaml
│   └── values.yaml
└── overlays/
    └── dev/
        ├── kustomization.yaml
        └── values-dev.yaml
```

## Fleet + Kustomize + Helm Workflow

### Step 1: Fleet Reads Git Repository
```yaml
# fleet.yaml
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: monitoring-dev
spec:
  repo: https://github.com/your-repo/gitops
  paths:
    - monitoring  # Fleet processes this directory
```

### Step 2: Kustomize Processes YAML
```yaml
# monitoring/base/kustomization.yaml
resources:
  - helmrelease.yaml

configMapGenerator:
  - name: monitoring-values
    files:
      - values.yaml
```

```yaml
# monitoring/overlays/dev/kustomization.yaml
resources:
  - ../../base

patches:
  - path: values-dev.yaml
    target:
      kind: Bundle
      name: monitoring
```

### Step 3: Fleet Creates Bundle
```yaml
# Fleet creates this Bundle from Kustomize output
apiVersion: fleet.cattle.io/v1alpha1
kind: Bundle
metadata:
  name: monitoring
spec:
  helm:
    chart: kube-prometheus-stack
    values:
      grafana:
        adminPassword: "dev123"  # From Kustomize patch
```

### Step 4: Fleet Deploys with Helm
Fleet uses Helm to install the chart with the values from Kustomize.

## Kustomize vs Helm: Key Differences

### Kustomize Approach

**What Kustomize Does:**
- **Declarative YAML Processing** - Transforms existing YAML files
- **Overlay System** - Base + overlays for different environments
- **Native Kubernetes** - Works with any Kubernetes resource
- **No Templates** - Uses patches and transformations
- **File Generation** - Creates ConfigMaps/Secrets from files

**Example Kustomize Configuration:**
```yaml
# base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml

configMapGenerator:
  - name: app-config
    files:
      - config.properties

# overlays/dev/kustomization.yaml
resources:
  - ../../base

patches:
  - path: replica-patch.yaml
    target:
      kind: Deployment
      name: my-app

configMapGenerator:
  - name: app-config
    behavior: merge
    files:
      - dev-config.properties
```

### Helm Approach

**What Helm Does:**
- **Template Engine** - Uses Go templates to generate YAML
- **Chart Packages** - Self-contained application packages
- **Values System** - External values override defaults
- **Release Management** - Tracks deployments with versions
- **Repository System** - Charts stored in repositories

**Example Helm Configuration:**
```yaml
# values.yaml
replicaCount: 1
image:
  repository: nginx
  tag: latest

# values-dev.yaml
replicaCount: 3
image:
  tag: dev
```

## When to Use Each Approach

### Use Kustomize When:
- ✅ **Simple YAML transformations** needed
- ✅ **Environment-specific patches** required
- ✅ **Native Kubernetes resources** (no Helm charts)
- ✅ **File-based configuration** (ConfigMaps from files)
- ✅ **Minimal learning curve** for Kubernetes users
- ✅ **GitOps workflows** with Fleet

### Use Helm When:
- ✅ **Complex applications** with many resources
- ✅ **Third-party charts** (Prometheus, Grafana, etc.)
- ✅ **Template-based configuration** needed
- ✅ **Release management** required
- ✅ **Chart repositories** available
- ✅ **Application packaging** needed

### Use Fleet + Kustomize + Helm When:
- ✅ **GitOps workflow** with Fleet
- ✅ **Environment-specific configurations** needed
- ✅ **Helm charts** with custom values
- ✅ **Complex multi-environment** deployments
- ✅ **Base + overlay** pattern desired

## Fleet's Integration Benefits

### 1. GitOps Workflow
```bash
# Fleet automatically:
# 1. Watches Git repository
# 2. Runs Kustomize on changes
# 3. Creates/updates Bundles
# 4. Deploys to target clusters
# 5. Uses Helm for chart installation
```

### 2. Environment Management
```yaml
# Single GitRepo can target multiple environments
spec:
  targets:
    - name: dev-clusters
      clusterSelector:
        matchLabels:
          env: dev
    - name: prod-clusters
      clusterSelector:
        matchLabels:
          env: prod
```

### 3. Kustomize + Helm Combination
```yaml
# Fleet Bundle with Helm chart and Kustomize-processed values
apiVersion: fleet.cattle.io/v1alpha1
kind: Bundle
metadata:
  name: monitoring
spec:
  helm:
    chart: kube-prometheus-stack
    values:
      grafana:
        adminPassword: "dev123"  # From Kustomize patch
      prometheus:
        enabled: true            # From Kustomize patch
```

## Real-World Example: Our Monitoring Setup

### What We Built:
```
monitoring/
├── base/
│   ├── kustomization.yaml      # Base configuration
│   ├── helmrelease.yaml        # Fleet Bundle
│   └── values.yaml             # Default values
└── overlays/
    └── dev/
        ├── kustomization.yaml  # Dev overlay
        └── values-dev.yaml     # Dev-specific patches
```

### How It Works:
1. **Fleet** watches the `monitoring` directory
2. **Kustomize** processes the base + overlay structure
3. **Fleet** creates a Bundle with the processed values
4. **Fleet** uses **Helm** to install the kube-prometheus-stack chart
5. **Fleet** deploys to clusters with `env: dev` label

### Benefits:
- ✅ **Environment-specific** Grafana passwords
- ✅ **Base configuration** shared across environments
- ✅ **Helm chart** benefits (versioning, rollbacks)
- ✅ **GitOps** workflow with Fleet
- ✅ **Kustomize** overlay pattern for environments

## Best Practices

### 1. Directory Structure
```
your-app/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patches/
│   └── prod/
│       ├── kustomization.yaml
│       └── patches/
└── fleet.yaml
```

### 2. Kustomize Configuration
```yaml
# Use patches for environment-specific changes
patches:
  - path: replica-patch.yaml
    target:
      kind: Deployment
      name: my-app

# Use ConfigMapGenerator for file-based configs
configMapGenerator:
  - name: app-config
    files:
      - config.properties
```

### 3. Fleet Configuration
```yaml
# Point to the root directory for Kustomize processing
spec:
  paths:
    - your-app  # Fleet will process Kustomize here
```

### 4. Helm Integration
```yaml
# Use Fleet Bundles for Helm charts
apiVersion: fleet.cattle.io/v1alpha1
kind: Bundle
spec:
  helm:
    chart: your-chart
    values:
      # Values from Kustomize processing
```

## Troubleshooting

### Kustomize Issues
```bash
# Test Kustomize locally
kubectl kustomize monitoring/overlays/dev

# Check Kustomize output
kubectl kustomize build monitoring/overlays/dev
```

### Fleet + Kustomize Issues
```bash
# Check Bundle configuration
kubectl get bundle <name> -n <namespace> -o yaml

# Check Kustomize processing in Fleet
kubectl describe gitrepo <name> -n fleet-local
```

### Helm Issues
```bash
# Check Helm release
helm list -n <namespace>

# Check Helm values
helm get values <release> -n <namespace>
```

## Summary

**Fleet + Kustomize + Helm** provides a powerful combination:
- **Fleet** handles GitOps workflow and cluster targeting
- **Kustomize** handles environment-specific configurations
- **Helm** handles application packaging and deployment

This combination gives you the best of all three tools:
- GitOps automation with Fleet
- Declarative configuration management with Kustomize
- Application packaging and release management with Helm

The key is understanding when to use each tool and how they complement each other in your deployment pipeline. 