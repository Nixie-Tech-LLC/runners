# runners

Self-hosted GitHub Actions runners for **nixiesoftware** — the org's own CI compute
("Mimir-as-runner-compute"). Split out of the Mimir platform repo into its own repo
(external-app-repo pattern, like `qadi` and `media-stack`); **Mimir references this repo** via a
Flux `GitRepository` + `Kustomization`.

## Layout

- **`deploy/`** — ARC (Actions Runner Controller) as HelmReleases: the controller + the `ci`
  (dind, for image builds) and `review` (lightweight validation) scale sets. Release names and
  namespaces are unchanged from the old Mimir definition so Flux **adopts the running releases in
  place** — no teardown, no CI blackout.
- **`runner-image/`** — a custom runner image (`ghcr.io/nixiesoftware/runner`): the upstream
  actions-runner plus docker CLI + buildx + git/jq/unzip, pre-baked to trim per-job setup.
- **`.github/workflows/build.yml`** — builds + publishes that image (on `runs-on: ci`).

## Cutover from Mimir (safe, no CI blackout)

All CI depends on these runners, so the move **adopts** rather than recreates:

1. **Mimir** adds `gitops/flux/apps/runners.yaml` (`GitRepository` → this repo + `Kustomization`
   → `./deploy`) **while keeping** `arc.yaml`. Flux reconciles the identical HelmReleases; the new
   `runners` Kustomization takes ownership (relabels) without changing anything.
2. Verify the `runners` Kustomization is Ready and the ARC controller + `ci`/`review` runners are
   still up.
3. **Mimir** removes `arc.yaml` + its `apps/kustomization.yaml` entry. Because the HelmReleases are
   now owned by `runners`, the `apps` prune leaves them alone.

## Switch the scale sets to the custom image

*After* the cutover is verified and the image is built:

1. Merge this repo → `build.yml` publishes `ghcr.io/nixiesoftware/runner:latest`.
2. Uncomment the `template.spec.containers[runner].image` block in `deploy/arc.yaml` (the `ci`
   scale set) — and add it to `review` if wanted.
3. Flux rolls the scale sets onto the pre-baked image.

Switching the image **before** it exists would `ImagePullBackOff` every runner (a CI blackout), so
the image change is deliberately decoupled from the ownership cutover.
