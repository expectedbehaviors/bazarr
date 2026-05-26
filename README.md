# Bazarr Helm Chart

Baseline Helm chart for [Bazarr](https://github.com/morpheus65535/bazarr) (subtitle manager for the *arr stack). Built on the [bjw-s app-template](https://github.com/bjw-s/helm-charts).

## What this baseline does

- Uses a sanitized baseline: no real hostnames or environment-specific PVC names.
- Expects Kubernetes secret **postgresql-credentials** (keys: `hostname`, `port`, `username`, `password`) for the Bazarr database. Create it manually or via **onepassworditem**.
- Optional 1Password sync: set `onepassworditem.enabled=true` and `onepassworditem.items` (e.g. `postgresql-credentials`, `bazarr`) so the operator creates/updates the secret(s) in the release namespace.
- Override persistence with `existingClaim` in your values for config, downloads, and media (e.g. same media PVCs as Plex).

## Secrets required

- **postgresql-credentials** (required for app startup): keys `hostname`, `port`, `username`, `password`. Bazarr uses database name `bazarr`. Provide via 1Password item + onepassworditem, or create the Secret yourself.
- **bazarr** (optional): app-specific keys; reference via onepassworditem if needed.

## Key values

| Area | Where | Default |
|------|-------|---------|
| Secret source | `onepassworditem.enabled` | `false` |
| 1Password items | `onepassworditem.items` | `[]` |
| Ingress | `bazarr.ingress.main.enabled` | `false` |
| Config storage | `bazarr.persistence.config.*` | PVC 2Gi |
| Downloads / media | `bazarr.persistence.downloads` / `media.*` | `emptyDir` (override with `existingClaim` in instantiation) |

## Install

From Helm repo (after chart is published):

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/bazarr
helm install bazarr expectedbehaviors/bazarr -f my-values.yaml -n bazarr --create-namespace
```

## Render & validation

```bash
helm dependency update . && helm template bazarr . -f values.yaml -n bazarr
```

Ensure secret **postgresql-credentials** exists in the namespace (or mock for template-only). With onepassworditem, the operator creates it from the referenced 1Password item.

## Publishing (maintainers)

This chart is published to **https://expectedbehaviors.github.io/bazarr** via GitHub Actions:

1. **Release bazarr chart on merge to main** — On push to `main` (or manual dispatch), lints the chart and creates a release with tag `bazarr-v<version>`.
2. **Helm chart publish** — Runs on that release (or manual dispatch) and publishes the package to the repo’s GitHub Pages. Pull from the repo with `helm repo add expectedbehaviors https://expectedbehaviors.github.io/bazarr`.
3. **Enable GitHub Pages** for the repo: **Settings → Pages → Source: Deploy from a branch** (or **GitHub Actions** if the publish action deploys to Pages). Branch: **gh-pages** (or as configured by the action).

## Argo CD

Point your Application at this repo (path: `.`) and pass your values. Namespace typically `bazarr`. Override persistence with `existingClaim` for config, downloads, and media to match your environment; set `onepassworditem.items` for DB and app secrets.
