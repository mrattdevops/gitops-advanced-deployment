# Engineering case study — GitOps Python delivery platform

## Problem

I wanted to move beyond manually applying Kubernetes manifests and create a repeatable deployment flow with a clear separation between building a container image and deciding which image an environment should run.

## Approach

I built a Python Flask API with pytest checks and containerized it using Docker. GitHub Actions performs CI and updates a separate GitOps repository. A Helm chart shares Kubernetes configuration across dev and prod. An Argo CD ApplicationSet generates two Applications: automated dev and manually synchronized prod. Metrics Server and HPA provide CPU-based scaling in dev; Prometheus and Grafana surface infrastructure and application metrics.

## Technical decisions

- Kept application code separate from GitOps state.
- Tagged images with Git commit SHAs for traceability.
- Used a single reusable Helm chart with per-environment values.
- Used PR-based promotion and manual Argo CD prod sync instead of automatically promoting every dev change.
- Kept resource-constrained monitoring settings suitable for a local K3s node.

## Failures and what they taught me

| Situation | Diagnosis and response | Lesson |
|---|---|---|
| Git push `403` | Wrong GitHub credentials or token permission | Verify identity and scoped access |
| Argo CD manifest error | Malformed YAML prevented rendering | Validate YAML/Helm before GitOps promotion |
| `ImagePullBackOff` | Placeholder or invalid tag | Publish and verify artifacts before deployment |
| CI tried localhost Kubernetes API | Runner lacked cluster access | Use offline schema checks instead of assuming kubeconfig |
| Rollout timed out | Deliberately invalid prod-like image | Restore known-good Git desired state and re-sync |
| HPA remained at max after load | Scale-down stabilization held prior recommendations | Distinguish desired replica calculation from instant scaling |

## Observed outcomes

- Dev and prod-like Applications reached `Synced / Healthy`.
- Prod-like deployment recovered after rollback with two pods ready.
- Dev HPA rose from one to four replicas during load, then returned to one.
- Prometheus/Grafana infrastructure dashboards were configured, followed by application `/metrics` instrumentation and ServiceMonitor work.

## Remaining engineering tasks

- Migrate manually installed monitoring Helm release to Argo CD safely, preserving Grafana credentials and dashboard definitions.
- Tighten GitHub Actions security, pin dependencies and action versions, and add branch protection.
- Review scanning order and manifest-list architecture coverage.
- Remove the CPU-intensive testing endpoint before any external exposure.
- Add alerting, secrets management, network isolation, GitOps bootstrap and recovery instructions.

## Portfolio framing

This is a **production-inspired lab on a local single-node Kubernetes cluster**, not a production deployment. Its value is the engineering process, release controls, demonstrated recovery, autoscaling experiment, and operational visibility.
