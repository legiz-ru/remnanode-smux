## remnanode-smux

Automated build of [Remnawave Node](https://github.com/remnawave/node) with the
[Jolymmiles/Xray-core](https://github.com/Jolymmiles/Xray-core) fork (smux support) baked in instead of the
upstream XTLS core.

This repository does **not** carry a copy of the node's source code. A scheduled GitHub Actions workflow
checks out the latest upstream release directly and builds it with a different Xray core — nothing about the
node itself is modified.

### Docker image

Published to GitHub Container Registry only (no Docker Hub), for `linux/amd64` and `linux/arm64`:

```bash
docker pull ghcr.io/legiz-ru/remnanode-smux:latest
```

```yaml
services:
    remnanode:
        image: ghcr.io/legiz-ru/remnanode-smux:latest
        container_name: remnanode
        hostname: remnanode
        network_mode: host
        restart: always
        environment:
            - NODE_PORT=2222
            - SECRET_KEY=""
```

See [docker-compose-prod.yml](./docker-compose-prod.yml) for the full example.

### Tags

Every published image is tagged `v<node-version>-<core-version>`, e.g. `v3.4.0-26.8.25-1457` for node `v3.4.0`
built with Xray core `v26.8.25-1457`. `:latest` always points at the most recently published combination.
Check the [packages page](https://github.com/legiz-ru/remnanode-smux/pkgs/container/remnanode-smux) or this
repo's [releases](https://github.com/legiz-ru/remnanode-smux/releases) for the full history — every publish
gets a release with the node and core versions linked to their upstream release pages.

### Bundled core vs. the runtime core loader

Since upstream `3.1.0` the node ships a `CoreLoaderService`: when the config pushed by the panel contains a
`core` section (`url` + `sha256`), the binary is downloaded at startup and `/usr/local/bin/rw-core` is
repointed at it — **the core baked into the image is then not used**. With no `core` section the service rolls
the symlink back to the bundled `/usr/local/bin/xray`, which is the Jolymmiles build from this image.

So keep the panel's `core` section empty to run the bundled smux core, or point it at a smux build yourself if
you prefer managing the core from the panel (note that the loader expects a raw binary, not the `.zip`
published in the Xray-core releases, and gives the download 15 seconds).

### How publishing works

[`.github/workflows/auto-release.yml`](./.github/workflows/auto-release.yml) runs every 3 hours (and on
manual dispatch):

1. Resolves the latest release of [`remnawave/node`](https://github.com/remnawave/node) and of
   [`Jolymmiles/Xray-core`](https://github.com/Jolymmiles/Xray-core) via each repo's `releases/latest`.
2. Computes the image tag `v<node>-<core>` and checks whether it's already published — if so, the run stops
   there, no build.
3. Verifies both Xray core release assets (`Xray-linux-64.zip`, `Xray-linux-arm64-v8a.zip`) actually resolve
   (HTTP 200) before building — Jolymmiles has occasionally published a tag before its assets finished
   uploading, or replaced a tag outright, so this check exists specifically to avoid building against a
   release that isn't really there yet.
4. Checks out `remnawave/node` at that release tag and builds `linux/amd64` + `linux/arm64` natively (no
   QEMU), passing the resolved core version as build args — the node's `Dockerfile` itself is untouched.
5. Pushes the manifest to `ghcr.io/legiz-ru/remnanode-smux:<tag>` and moves `:latest`, then creates a GitHub
   release with both upstream links and the pull command.

There is no manual review step for either upstream — both node and core releases go out automatically once
their assets check out. Trigger a specific combination on demand via **Actions → Auto Release → Run
workflow**, optionally overriding `node_ref` and/or `core_version`.

### Credits

Based on [remnawave/node](https://github.com/remnawave/node) — check its
[open issues](https://github.com/remnawave/panel/issues) to help the upstream project.
