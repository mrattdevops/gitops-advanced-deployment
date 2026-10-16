# Portfolio evidence checklist

Add actual screenshots (with timestamps or commit IDs when useful) to `evidence/`, then embed them in the README.

- [ ] GitHub Actions successful build/test/scan/push job (mask secrets)
- [ ] Docker Hub image tags showing SHA traceability
- [ ] Argo CD ApplicationSet with `python-api-dev` and `python-api-prod`
- [ ] Both Applications `Synced / Healthy`
- [ ] Helm dev/prod values or diffs showing environment distinction
- [ ] Production promotion PR showing tag-only change
- [ ] Rollback Git commit and recovered `2/2` rollout
- [ ] HPA max of 4 during CPU load and return to 1 at idle
- [ ] Grafana HPA replica time series
- [ ] Grafana Flask request rate and P95 latency panels
- [ ] Prometheus target state `UP` for Python API ServiceMonitor

Recommended filenames: `ci-success.png`, `argocd-apps.png`, `hpa-scale.png`, `grafana-python-api.png`.

Do not upload credentials, tokens, private IP addresses that need to remain internal, or sensitive environment variables. Redact before publishing.
