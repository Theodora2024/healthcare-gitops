# Release Notes

## Healthcare application release — 10 October 2026

### Release image tag

`a496e9c`

### Container images

- Backend: `245758552830.dkr.ecr.eu-west-2.amazonaws.com/healthcare-prod-backend:a496e9c`
- Frontend: `245758552830.dkr.ecr.eu-west-2.amazonaws.com/healthcare-prod-frontend:a496e9c`

### Verified deployment state

- Development: 1 backend and 1 frontend replica ready.
- Staging: 2 backend and 2 frontend replicas ready.
- Production: 2 backend and 2 frontend replicas ready.
- PostgreSQL StatefulSets: ready in dev, stage and prod.
- PostgreSQL persistent volume claims: `Bound`, 5 GiB, `gp3` in each GitOps environment.
- Metrics Server: EKS add-on status `ACTIVE`, version `v0.9.0-eksbuild.11`.
- HPA CPU metrics: available and showing numeric readings.
- Argo CD: dev and stage `Synced/Healthy`; production `OutOfSync/Healthy`.

### Outstanding verification

1. Investigate why the production Argo CD application is `OutOfSync`.
2. Resolve and verify DNS for the application and Argo CD hostnames.
3. Verify public URL responses after DNS resolution works.
4. Ensure the Metrics Server add-on is represented in the Terraform configuration.

### Release commit

Record the Git commit used for this release with:

```bash
git log -1 --format='%H %cI %s'
```

The image tag is the container release identifier; it is not necessarily the Git commit for the manifests.
