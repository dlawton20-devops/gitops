# Fleet Troubleshooting Guide

This guide contains all the essential Fleet commands for troubleshooting and monitoring your Fleet deployments.

## GitRepo Management

### Check GitRepo Status
```bash
# Get all GitRepos
kubectl get gitrepo -n fleet-local

# Describe specific GitRepo
kubectl describe gitrepo <gitrepo-name> -n fleet-local

# Get GitRepo YAML
kubectl get gitrepo <gitrepo-name> -n fleet-local -o yaml
```

### Force GitRepo Refresh
```bash
# Remove commit annotation to force refresh
kubectl annotate gitrepo <gitrepo-name> -n fleet-local fleet.cattle.io/commit- --overwrite

# Delete and recreate GitRepo
kubectl delete gitrepo <gitrepo-name> -n fleet-local
kubectl apply -f fleet.yaml
```

### GitRepo Status Indicators
- **Ready: True** - GitRepo is processing successfully
- **Ready: False** - Issues with GitRepo processing
- **GitPolling: True** - Git polling is active
- **Reconciling: True** - GitRepo is being reconciled

## Bundle Management

### Check Bundle Status
```bash
# Get all Bundles
kubectl get bundles -A

# Get Bundles in specific namespace
kubectl get bundles -n <namespace>

# Describe specific Bundle
kubectl describe bundle <bundle-name> -n <namespace>

# Get Bundle YAML
kubectl get bundle <bundle-name> -n <namespace> -o yaml
```

### Bundle Status Indicators
- **Ready: True** - Bundle is ready
- **Ready: False** - Issues with Bundle configuration
- **Display.readyClusters: X/Y** - Number of clusters where Bundle is ready

## BundleDeployment Management

### Check BundleDeployment Status
```bash
# Get all BundleDeployments
kubectl get bundledeployments -A

# Get BundleDeployments in specific namespace
kubectl get bundledeployments -n <namespace>

# Describe specific BundleDeployment
kubectl describe bundledeployment <bundledeployment-name> -n <namespace>
```

### BundleDeployment Status Indicators
- **DEPLOYED: True** - Successfully deployed to cluster
- **MONITORED: True** - Fleet is monitoring the deployment
- **STATUS: Ready** - Deployment is ready
- **STATUS: Modified** - Changes detected, not yet applied
- **STATUS: Error** - Deployment failed

## Cluster Management

### Check Cluster Status
```bash
# Get all clusters
kubectl get clusters -A --show-labels

# Get clusters in specific namespace
kubectl get clusters -n <namespace> --show-labels

# Describe specific cluster
kubectl describe cluster <cluster-name> -n <namespace>
```

### Cluster Status Indicators
- **Ready: True** - Cluster is ready
- **Ready: False** - Issues with cluster
- **Imported: True** - Cluster has been imported
- **Processed: True** - Cluster has been processed

## Helm Integration

### Check Helm Releases
```bash
# List Helm releases
helm list -A

# List Helm releases in namespace
helm list -n <namespace>

# Get Helm release status
helm status <release-name> -n <namespace>

# Get Helm release values
helm get values <release-name> -n <namespace>
```

### Helm Release Status
- **STATUS: deployed** - Successfully deployed
- **STATUS: failed** - Deployment failed
- **STATUS: pending** - Deployment in progress

## Resource Monitoring

### Check Resources in Namespace
```bash
# Get all resources
kubectl get all -n <namespace>

# Get specific resource types
kubectl get pods -n <namespace>
kubectl get services -n <namespace>
kubectl get configmaps -n <namespace>
kubectl get secrets -n <namespace>
```

### Check Resource Events
```bash
# Get events in namespace
kubectl get events -n <namespace>

# Get events sorted by time
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

## Common Troubleshooting Scenarios

### 1. GitRepo Not Picking Up Changes
```bash
# Check current commit
kubectl describe gitrepo <gitrepo-name> -n fleet-local | grep "Commit:"

# Force refresh
kubectl annotate gitrepo <gitrepo-name> -n fleet-local fleet.cattle.io/commit- --overwrite

# Delete and recreate
kubectl delete gitrepo <gitrepo-name> -n fleet-local
kubectl apply -f fleet.yaml
```

### 2. Bundle Not Deploying
```bash
# Check Bundle status
kubectl describe bundle <bundle-name> -n <namespace>

# Check BundleDeployment status
kubectl get bundledeployments -A | grep <bundle-name>

# Check cluster targeting
kubectl get clusters -A --show-labels
```

### 3. Helm Chart Not Installing
```bash
# Check Helm release status
helm status <release-name> -n <namespace>

# Check Helm values
helm get values <release-name> -n <namespace>

# Check Bundle values
kubectl get bundle <bundle-name> -n <namespace> -o yaml | grep -A 10 "values:"
```

### 4. CRD Issues
```bash
# Check if CRDs are installed
kubectl get crd | grep fleet

# Check specific CRD
kubectl get crd gitrepos.fleet.cattle.io
kubectl get crd bundles.fleet.cattle.io
kubectl get crd bundledeployments.fleet.cattle.io
```

## Logs and Debugging

### Check Fleet Controller Logs
```bash
# Get Fleet controller pods
kubectl get pods -n cattle-fleet-system

# Check Fleet controller logs
kubectl logs -n cattle-fleet-system deployment/fleet-controller

# Follow logs
kubectl logs -n cattle-fleet-system deployment/fleet-controller -f
```

### Check Fleet Agent Logs
```bash
# Get Fleet agent pods
kubectl get pods -n cattle-fleet-system

# Check Fleet agent logs
kubectl logs -n cattle-fleet-system deployment/fleet-agent

# Follow logs
kubectl logs -n cattle-fleet-system deployment/fleet-agent -f
```

### Check Git Job Logs
```bash
# Get Git job pods
kubectl get pods -n fleet-local

# Check Git job logs
kubectl logs -n fleet-local job/<gitjob-name>
```

## Configuration Validation

### Validate Fleet Configuration
```bash
# Check Fleet configuration
kubectl get gitrepo -n fleet-local -o yaml

# Validate Bundle configuration
kubectl get bundle -n <namespace> -o yaml

# Check cluster selectors
kubectl get clusters -A --show-labels
```

### Validate Helm Configuration
```bash
# Validate Helm chart
helm template <chart-name> <repo-url> --values <values-file>

# Dry run Helm install
helm install <release-name> <chart-name> --dry-run --values <values-file>
```

## Cleanup Commands

### Remove Fleet Resources
```bash
# Delete GitRepo
kubectl delete gitrepo <gitrepo-name> -n fleet-local

# Delete Bundle
kubectl delete bundle <bundle-name> -n <namespace>

# Delete BundleDeployment
kubectl delete bundledeployment <bundledeployment-name> -n <namespace>

# Delete Helm release
helm uninstall <release-name> -n <namespace>
```

### Clean Up Namespaces
```bash
# Delete namespace (will delete all resources)
kubectl delete namespace <namespace>

# Force delete namespace
kubectl delete namespace <namespace> --force --grace-period=0
```

## Best Practices

1. **Always check GitRepo status first** - Most issues start with GitRepo not picking up changes
2. **Verify cluster targeting** - Ensure clusters have the correct labels
3. **Check Bundle configuration** - Validate Bundle YAML structure
4. **Monitor BundleDeployment status** - This shows actual deployment status
5. **Use Helm commands for chart-specific issues** - Fleet uses Helm under the hood
6. **Check logs for detailed error messages** - Fleet controller and agent logs contain detailed information

## Quick Reference

### Status Check Commands
```bash
# Quick status overview
kubectl get gitrepo,bundle,bundledeployment -A

# Check specific deployment
kubectl get bundledeployment -A | grep <name>

# Check Helm releases
helm list -A
```

### Force Refresh Commands
```bash
# Force GitRepo refresh
kubectl annotate gitrepo <name> -n fleet-local fleet.cattle.io/commit- --overwrite

# Delete and recreate
kubectl delete gitrepo <name> -n fleet-local && kubectl apply -f fleet.yaml
```

This guide covers the most common Fleet troubleshooting scenarios and commands. Use these commands systematically to identify and resolve issues with your Fleet deployments. 