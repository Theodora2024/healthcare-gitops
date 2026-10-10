# Promotion and Rollback Guide

## Promotion workflow

Promote only a tested, identifiable image version. Use the same release tag across environments when that is the chosen release strategy.

1. Build and test backend and frontend images.
2. Push images to Amazon ECR and record their immutable tag or digest.
3. Update the image tag in the development overlay.
4. Commit and push the change.
5. Confirm Argo CD sync and workload readiness in development.
6. Promote the tested version to staging and repeat the checks.
7. Promote to production only after staging verification and the required approval.
8. Verify image versions, replicas, ingress, application health and database readiness.

Before promotion, render the manifests:

```bash
kubectl kustomize overlays/dev
kubectl kustomize overlays/stage
kubectl kustomize overlays/prod
```

## Rollback a manifest change

From a local clone of this repository:

1. Identify the commit that introduced the unwanted change:

   ```bash
   git log --oneline --decorate -10
   ```

2. Review the change before reverting it:

   ```bash
   git show <bad-commit-sha>
   ```

3. Revert the specific commit:

   ```bash
   git revert <bad-commit-sha>
   ```

4. Push the revert:

   ```bash
   git push origin main
   ```

5. Monitor Argo CD and confirm the affected workload returns to the intended image and replica state.

If several later changes depend on the faulty commit, review the dependency chain before reverting. Avoid force-pushing shared release history.

## Roll back to a previous image

Change the relevant overlay's image tag to a previously tested release, review the rendered manifests, and commit and push the change. Wait for Argo CD reconciliation, then verify rollout health and the actual image used by running pods.

Use a known-good tag or digest. Do not assume that `latest` refers to the previous working release.

## Database caution

Rolling back Kubernetes manifests or container images does not automatically reverse database schema migrations or data changes. Back up important data and follow a tested database recovery plan before any schema rollback.

## Current project note

The production Argo CD application was `OutOfSync/Healthy` at the last verification. Investigate and review the diff before using a sync or rollback as a corrective action. Do not force-sync or delete resources without understanding the difference.
