# homelab-ci — reusable CI/CD workflow

A single **reusable GitHub Actions workflow** that every app repo calls to:

1. **Build & test** the app
2. **Build** a Docker image and **scan** it (Trivy)
3. **Push** to **GHCR** with an immutable tag (`sha-<short-sha>`)
4. Open an **auto-PR** to [`homelab-workloads`](https://github.com/acid0ikario/homelab-workloads)
   bumping the image tag — merge it and ArgoCD deploys.

Part of the [homelab](https://github.com/acid0ikario/homelab) project.

## Use it from an app repo

Create `.github/workflows/deploy.yml` in your app repo:

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  ci:
    uses: acid0ikario/homelab-ci/.github/workflows/build-and-deploy.yml@main
    permissions:
      contents: read
      packages: write
    with:
      image-name: garmindashboard          # -> ghcr.io/acid0ikario/garmindashboard
      workload-path: workloads/garmindashboard   # path in homelab-workloads
    secrets:
      GITOPS_TOKEN: ${{ secrets.GITOPS_TOKEN }}   # PAT with repo scope on homelab-workloads
```

See [`examples/app-deploy.yml`](examples/app-deploy.yml) for a full example.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-name` | yes | — | Image name under `ghcr.io/acid0ikario/` |
| `workload-path` | yes | — | Path in `homelab-workloads` whose `kustomization.yaml` gets bumped |
| `gitops-repo` | no | `acid0ikario/homelab-workloads` | GitOps repo to open the PR against |
| `dockerfile` | no | `Dockerfile` | Path to the Dockerfile |
| `context` | no | `.` | Docker build context |
| `run-tests` | no | `true` | Whether to run the test step |

## Secrets

| Secret | Description |
|--------|-------------|
| `GITOPS_TOKEN` | A PAT (or fine-grained token) with **contents: write** + **pull-requests: write** on `homelab-workloads`, used to push the branch and open the PR. `GITHUB_TOKEN` can't open PRs across repos. |

## Why auto-PR instead of writing the tag directly?

The PR is an **auditable checkpoint** in Git: you see exactly which image is going
to production, can require review/approval, and get a clean deploy history. ArgoCD
only acts once the PR is merged.

## License

MIT © acid0ikario
