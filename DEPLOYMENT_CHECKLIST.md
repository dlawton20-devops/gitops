# Quick Deployment Checklist

## Pre-Deployment
- [ ] Fleet installed and running
- [ ] Git repository forked/cloned
- [ ] Repository URL updated in fleet.yaml files
- [ ] Branch names configured correctly
- [ ] Cluster labels set (`env=dev` or similar)

## Deployment Commands
```bash
# 1. Check Fleet status
kubectl get crd | grep fleet

# 2. Check cluster labels
kubectl get clusters.fleet.cattle.io -A --show-labels

# 3. Apply Fleet GitRepo
kubectl apply -f fleet.yaml

# 4. Verify deployment
kubectl get gitrepo -A
kubectl get bundle -A
kubectl get pods -n monitoring
```

## Post-Deployment Verification
- [ ] GitRepo status shows "Ready"
- [ ] Bundle status shows "Ready"
- [ ] BundleDeployment status shows "Deployed"
- [ ] Monitoring pods are running
- [ ] Grafana accessible via port-forward
- [ ] Prometheus accessible via port-forward

## Quick Access Commands
```bash
# Grafana
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring

# Prometheus  
kubectl port-forward svc/prometheus-stack-kube-prometheus-prometheus 9090:9090 -n monitoring

# Alertmanager
kubectl port-forward svc/prometheus-stack-kube-prometheus-alertmanager 9093:9093 -n monitoring
```

## Troubleshooting Commands
```bash
# Check Fleet status
kubectl describe gitrepo monitoring-dev -n fleet-local
kubectl describe bundle monitoring-dev-monitoring -n fleet-local

# Check monitoring status
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl describe pod <pod-name> -n monitoring

# Reset if needed
kubectl delete gitrepo --all -n fleet-local
kubectl delete namespace monitoring
kubectl apply -f fleet.yaml
``` 