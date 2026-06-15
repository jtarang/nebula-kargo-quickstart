# Kargo Quickstart — Akuity Onboarding

Fork of Akuity's [Kargo Quickstart](https://docs.akuity.io/tutorials/kargo-quickstart/).

## Design
- A new image tag → Kargo Warehouse creates Freight.
- Promoting a Stage renders the manifests (`kustomize build`) and commits them to that env's
  branch (`env/<env>`); Argo CD applies what's in git.

| Path | Purpose |
|------|---------|
| `akuity/` | Argo CD instance, cluster, guestbook `ApplicationSet` |
| `app/` | guestbook base + dev/staging/prod kustomize overlays |
| `kargo/` | Kargo Project, Warehouse, Stages, PromotionTask |

## My change: reusable `PromotionTask`
The three Stages were ~95% duplicated YAML (same 7-step pipeline, only the env name differed).
I extracted it into one `PromotionTask` (`promote-guestbook`) parameterized by `env`; each
Stage just calls it. `stages.yaml`: **182 → 56 lines**.
- **Win:** single source of truth, modify the pipeline once, no drift between stages.
- **Tradeoff:** one layer of indirection; per stage logic now lives in the task file.
- **Gotcha:** inside a task, step outputs must use `task.outputs.*` (not plain `outputs.*`).

## Deployment
- Workload is GitOps-managed (ApplicationSet tracks `env/*` branches).
- Control plane is applied imperatively: `akuity argocd apply -f akuity/` and
  `kargo apply -f kargo/`. **Merging to `main` does not auto-update Kargo**.

## Assumptions
- Public Git repo + public GHCR image (unauthenticated read).
- Access to a Akuity Platform
