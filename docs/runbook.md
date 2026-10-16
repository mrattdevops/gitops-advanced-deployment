# Operations runbook

## Prerequisites

`kubectl`, Helm, Rancher Desktop K3s, Argo CD in `argocd`, deployed Helm chart, valid registry images, and read/write Git credentials where needed. Check the Kubernetes context before any action.

```bash
kubectl config current-context
kubectl config use-context rancher-desktop
kubectl get nodes -o wide
kubectl get applications -n argocd
```

## Deployment verification

```bash
kubectl get pods,deploy,svc -n gitops-dev
kubectl get pods,deploy,svc -n gitops-prod
kubectl get hpa -n gitops-dev
kubectl get deployment python-api-dev -n gitops-dev -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
kubectl get deployment python-api-prod -n gitops-prod -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

## Helm validation (from deployment repo)

```bash
helm lint charts/python-api -f environments/dev/values.yaml
helm lint charts/python-api -f environments/prod/values.yaml
helm template python-api-dev charts/python-api -n gitops-dev -f environments/dev/values.yaml >/tmp/gitops-dev.yaml
helm template python-api-prod charts/python-api -n gitops-prod -f environments/prod/values.yaml >/tmp/gitops-prod.yaml
```

## App health checks

```bash
kubectl -n gitops-dev port-forward svc/python-api-dev 8080:80
# second shell
curl -fsS http://localhost:8080/health/ready
curl -fsS http://localhost:8080/api/info
curl -fsS http://localhost:8080/metrics | head
```

## Metrics / HPA diagnostics

```bash
kubectl top nodes
kubectl top pods -n gitops-dev
kubectl describe hpa python-api-dev -n gitops-dev
kubectl get hpa python-api-dev -n gitops-dev -w
```

## Prometheus / Grafana

```bash
kubectl get pods -n monitoring
kubectl get servicemonitors -A
kubectl get prometheus -n monitoring
kubectl -n monitoring port-forward svc/monitoring-grafana 3000:80
```

Avoid committing Grafana admin passwords. Existing monitoring resources were created through manual Helm and should not be blindly adopted by Argo CD.

## Troubleshooting matrix

| Symptom | Inspect | Typical direction |
|---|---|---|
| CI cannot push config | GitHub token scope, branch rules | Grant minimum Contents write to correct repo |
| Trivy blocks delivery | Vulnerabilities and base image | Upgrade dependency/base image, review exception policy |
| Argo CD `ComparisonError` | Application conditions, Helm template | Fix YAML / missing values / values-file paths |
| Pod `ImagePullBackOff` | Pod events, exact image/tag | Verify registry access and image existence |
| Rollout timeout | `kubectl describe deploy`, pods, ReplicaSets | Restore last known-good tag via GitOps |
| HPA `<unknown>` | Metrics Server, `kubectl top`, CPU requests | Fix metric availability |
| High replica count at low CPU | `kubectl describe hpa` | Check scale-down stabilization, recent recommendations |
| Grafana `No data` | Prometheus targets, ServiceMonitor selectors | Check namespace/label selectors and scrape endpoints |

## Useful commands

```bash
kubectl -n gitops-prod describe deployment python-api-prod
kubectl -n gitops-prod get rs
kubectl -n gitops-prod get events --sort-by=.lastTimestamp | tail -20
kubectl -n argocd get application python-api-prod -o yaml
kubectl -n monitoring top pods
```
