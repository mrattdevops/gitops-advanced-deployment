# Release promotion and rollback

## Routine delivery

1. Source commit triggers GitHub Actions tests, image build and vulnerability scan.
2. CI publishes a SHA-tagged image to Docker Hub, then updates dev Helm values in the deployment repo.
3. Argo CD dev auto-syncs; verify readiness, endpoint behavior and Grafana metrics.
4. Create a Git PR updating **only prod's image repository/tag** to the tested dev artifact. Preserve prod replicas, environment values and manual sync policy.
5. Merge after review, inspect `python-api-prod` diff in Argo CD, then manually Sync.
6. Confirm prod `Synced / Healthy` and deployment `2/2` ready.

## Example inspection commands

```bash
kubectl -n gitops-dev get deploy python-api-dev -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
kubectl -n gitops-prod get deploy python-api-prod -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
helm lint charts/python-api -f environments/prod/values.yaml
```

## Rollback exercise performed

In the local prod-like namespace, an intentionally nonexistent tag was promoted; the rollout timed out. The last known-good image tag (`36f432a`, as identified in the lab) was restored through Git, prod was manually synchronized, and the Kubernetes Deployment returned to `2/2` ready with Argo CD `Synced / Healthy`.

## Rollback procedure

1. Confirm last known-good image (Deployment's prior ReplicaSet and Git history).
2. Create a rollback branch and PR changing the prod tag back to the known-good value.
3. Merge and manually synchronize prod in Argo CD.
4. Verify:

```bash
kubectl -n gitops-prod rollout status deployment/python-api-prod --timeout=180s
kubectl -n gitops-prod get pods
kubectl -n argocd get applications
```

Do not rely on `kubectl rollout undo` as the lasting fix; the GitOps desired state must also change. Do not perform destructive failure testing on real production infrastructure.
