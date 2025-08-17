# Build and Push Docker Image Action

A GitHub Action to build and push Docker images with multi-platform support and caching.  
It tags images based on the git tag and only pushes `:latest` for semantic version tags (`X.Y.Z`).

---

## Features

- Build and push Docker images to any registry (default: `ghcr.io`)
- Multi-platform builds (default: `linux/amd64,linux/arm64`)
- Push `:latest` tag only for semantic version tags
- Incremental build caching via registry
- Fully reusable as a composite action

---

## Inputs

| Input | Description | Default | Required |
|-------|-------------|---------|----------|
| `github-token` | GitHub token with registry write access | — | Yes |
| `context` | Docker build context | `.` | No |
| `dockerfile` | Path to Dockerfile | `Dockerfile` | No |
| `registry` | Container registry | `ghcr.io` | No |
| `image-name` | Name of the Docker image | — | Yes |
| `platforms` | Comma-separated list of target platforms | `linux/amd64,linux/arm64` | No |

---

## Outputs

| Output | Description |
|--------|------------|
| `tag` | The Docker tag that was pushed |
| `latest` | `"true"` if `:latest` was also pushed, otherwise `"false"` |

---

## Usage Examples

### 1. Standard workflow for semantic version tags

```yaml
name: Release Docker Image

on:
  release:
    types: [created]
  workflow_dispatch:
  push:
    tags:
      - '[0-9]+.[0-9]+.[0-9]+-dev'
      - '[0-9]+.[0-9]+.[0-9]+-dev.[0-9]+'
      - '[0-9]+.[0-9]+.[0-9]+-rc'
      - '[0-9]+.[0-9]+.[0-9]+-rc.[0-9]+'
      - '[0-9]+.[0-9]+.[0-9]+'

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Build & Push Docker
        uses: ./.github/actions/build-push-docker
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          image-name: ${{ github.repository }}
```

### 2. Build multi-platform images with caching
```yaml
- name: Build & Push Multi-Platform
  uses: ./.github/actions/build-push-docker
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    image-name: my-org/my-app
    context: ./docker
    dockerfile: ./docker/Dockerfile
    platforms: linux/amd64,linux/arm64,linux/arm/v7
```

### 3. Example: Only tag :latest for semantic version
If the pushed tag is 1.2.3, the action will create:
```bash
ghcr.io/my-org/my-app:1.2.3
ghcr.io/my-org/my-app:latest
```

If the pushed tag is 1.2.3-rc.1, the action will create:
```bash
ghcr.io/my-org/my-app:1.2.3-rc.1
```

### Notes
- Use github-token with write permissions to the registry.
- Caching requires a registry that supports --cache-from and --cache-to.
- Works with Docker Hub, GHCR, and other OCI-compliant registries.
