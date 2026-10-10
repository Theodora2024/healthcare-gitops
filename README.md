# Healthcare GitOps on Amazon EKS

## Project overview

This project demonstrates GitOps-based deployment and environment promotion for a containerized healthcare application on Amazon EKS. Kubernetes manifests are managed in Git and reconciled by Argo CD.

## Architecture

- **Amazon EKS:** Managed Kubernetes cluster in `eu-west-2`.
- **Amazon ECR:** Stores the backend and frontend container images.
- **Argo CD:** Continuously reconciles Git manifests with cluster resources.
- **Kustomize:** Maintains a shared base and environment-specific overlays.
- **AWS Load Balancer Controller:** Provisions and manages Application Load Balancer ingress.
- **ExternalDNS:** Manages DNS records when its permissions and DNS configuration are correctly configured.
- **External Secrets Operator:** Supports integration with external secret stores.
- **Metrics Server and HPA:** Expose CPU/memory metrics and enable CPU-based horizontal scaling.
- **PostgreSQL:** Runs as a StatefulSet with persistent EBS-backed storage in each GitOps environment.

## Environments

| Environment | Namespace | Backend replicas | Frontend replicas |
|---|---|---:|---:|
| Development | `healthcare-dev` | 1 | 1 |
| Staging | `healthcare-stage` | 2 | 2 |
| Production | `healthcare-prod` | 2 | 2 |

The backend and frontend in all three environments were verified running the image tag `a496e9c` during the final verification on 10 October 2026. Replica counts may change through scaling or future releases.

## Repository structure

```text
.
├── base/                  # Shared Kubernetes resources
├── overlays/
│   ├── dev/               # Development configuration
│   ├── stage/             # Staging configuration
│   └── prod/              # Production configuration
├── bootstrap/             # Argo CD bootstrap/application resources
├── dev-rendered.yaml      # Previously rendered development manifests
├── stage-rendered.yaml    # Previously rendered staging manifests
└── prod-rendered.yaml     # Previously rendered production manifests
```

## Deployment workflow

1. Build and test the backend and frontend images.
2. Push versioned images to Amazon ECR.
3. Update the image tag in the target Kustomize overlay.
4. Commit and push the manifest change to Git.
5. Argo CD detects the change and reconciles the target environment.
6. Verify application health, replica readiness, image versions, ingress and database storage before promoting to the next environment.

Use immutable release tags rather than relying on `latest`.

## Useful commands

Render an environment before deployment:

```bash
kubectl kustomize overlays/dev
kubectl kustomize overlays/stage
kubectl kustomize overlays/prod
```

Inspect GitOps applications and workloads:

```bash
kubectl get applications.argoproj.io -n argocd
kubectl get deployments,pods -n healthcare-dev
kubectl get deployments,pods -n healthcare-stage
kubectl get deployments,pods -n healthcare-prod
kubectl get hpa -A
kubectl top nodes
kubectl get ingress -A
kubectl get statefulsets,pvc -A
```

## Security and operations

- Do not commit credentials, tokens, or real secret values to Git.
- Use managed secret storage and least-privilege IAM permissions.
- Review rendered manifests before merging changes.
- Verify readiness, database persistence, ingress and DNS before declaring a release successful.
- Keep Terraform infrastructure configuration aligned with manually installed EKS add-ons.

## Current verification notes

At the last recorded check, dev and stage were `Synced/Healthy` in Argo CD. Production was `OutOfSync/Healthy`; its pods were running, but the synchronization discrepancy still requires investigation.

Metrics Server was `ACTIVE` at version `v0.9.0-eksbuild.11`, and CPU metrics were available to the HPAs.

Ingress resources existed for the application and Argo CD, but DNS lookups for their hostnames failed from AWS CloudShell during verification. Public URL reachability and DNS therefore remain unverified.

See [RELEASE_NOTES.md](RELEASE_NOTES.md), [docs/final-verification.md](docs/final-verification.md), and [docs/promotion-and-rollback.md](docs/promotion-and-rollback.md).
