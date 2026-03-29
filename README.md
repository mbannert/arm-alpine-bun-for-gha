# arm-alpine-bun-for-gha

A minimal ARM64 Alpine image with [Bun](https://bun.sh) and `git` pre-installed, built for use in GitHub Actions.

Hosted on GHCR: `ghcr.io/opentsi/bun-alpine-git`

## Why

GitHub's JavaScript-based actions (`actions/checkout`, `actions/setup-node`) do not work in Alpine containers on ARM64 runners. This image sidesteps that by pre-installing `git` so you can clone manually, and using Bun as a full Node.js drop-in — no Node installation needed.

The image is rebuilt weekly to pick up Alpine security patches even when the Bun version hasn't changed. [Renovate](https://docs.renovatebot.com) auto-merges Bun version bumps.

## Usage

### Deploy an Astro site to GitHub Pages

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-24.04-arm
    # Pin to a specific version. Check https://github.com/opentsi/bun-alpine-git/pkgs/container/bun-alpine-git
    # for the latest release. In production, use Renovate to keep this up to date automatically.
    container: ghcr.io/opentsi/bun-alpine-git:1.3.11
    steps:
      - name: Checkout
        run: git clone --depth 1 https://x-access-token:${{ github.token }}@github.com/${{ github.repository }}.git .

      - name: Install dependencies
        run: bun install --frozen-lockfile

      - name: Build
        run: bun run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

> The `deploy` job runs on plain `ubuntu-latest` — `actions/deploy-pages` is a JavaScript action and must run outside the Alpine container.

## Tags

| Tag | Description |
|-----|-------------|
| `latest` | Always the most recent build |
| `1.3.11` | Pinned to Bun 1.3.11 |

## Image contents

- Base: `oven/bun:1.3.11-alpine`
- Added: `git`, `tar`
