# Final Verification Report

**Verification date:** 10 October 2026  
**Cluster:** `eks-infra-prod`  
**AWS region:** `eu-west-2`

## 1. Argo CD

| Application | Sync status | Health status |
|---|---|---|
| `bootstrap` | Synced | Healthy |
| `healthcare-dev` | Synced | Healthy |
| `healthcare-stage` | Synced | Healthy |
| `healthcare-prod` | Synced | Healthy |

 Investigate the Argo CD diff before declaring all environments fully reconciled.

## 2. Workload readiness

| Namespace | Backend ready | Frontend ready | PostgreSQL |
|---|---:|---:|---|
| `healthcare-dev` | 1/1 | 1/1 | Running, 1/1 |
| `healthcare-stage` | 2/2 | 2/2 | Running, 1/1 |
| `healthcare-prod` | 2/2 | 2/2 | Running, 1/1 |

All listed application pods were `Running` and ready at the time of verification.

## 3. Container image versions

All three environments used these images with tag `a496e9c`:

- `245758552830.dkr.ecr.eu-west-2.amazonaws.com/healthcare-prod-backend:a496e9c`
- `245758552830.dkr.ecr.eu-west-2.amazonaws.com/healthcare-prod-frontend:a496e9c`

## 4. Metrics Server and autoscaling

The EKS Metrics Server add-on reported:

- Status: `ACTIVE`
- Version: `v0.9.0-eksbuild.11`
- Health issues: none reported

The Metrics API reported `AVAILABLE=True`. `kubectl top nodes` and `kubectl top pods -A` returned CPU and memory metrics. All six GitOps HPAs showed numeric CPU readings against their 70% targets.

## 5. PostgreSQL persistence

The PostgreSQL StatefulSets in `healthcare-dev`, `healthcare-stage` and `healthcare-prod` were ready at 1/1.

Each environment's PostgreSQL PVC was `Bound` with a capacity of 5 GiB and storage class `gp3`.

A separate legacy `healthcare` namespace also contains PostgreSQL and an older ingress. It is outside the three GitOps environment namespaces and should be considered separately during cleanup or migration planning.

## 6. Ingress and DNS

The following ingress hostnames were present and pointed to the shared ALB address in the Kubernetes ingress listing:

- `dev-healthcare.isidorah.site`
- `stage-healthcare.isidorah.site`
- `healthcare.isidorah.site`
- `argocd.isidorah.site`

However, `curl` from AWS CloudShell returned `Could not resolve host` for all four hostnames. DNS resolution and public application reachability are **not verified**. The ingress resources existing in Kubernetes does not, by itself, prove that DNS records resolve or that HTTPS/TLS is configured correctly.

## 7. Remaining actions

- Investigate DNS records, hosted zone configuration and name-server delegation.
- Re-run DNS and HTTP/HTTPS checks after resolution is restored.
- Reconcile the manually installed Metrics Server add-on with Terraform configuration.
- Capture a fresh verification report after these outstanding actions are complete.
