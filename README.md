## Remnanode smux

Fork of [Remnawave Node](https://github.com/remnawave/node) `3.1.1`, built with the
[Jolymmiles/Xray-core](https://github.com/Jolymmiles/Xray-core) fork (smux support) instead of the
upstream XTLS core.

Learn more about Remnawave Panel [here](https://docs.rw/).

### Docker image

Images are published to GitHub Container Registry only (no Docker Hub), for `linux/amd64` and `linux/arm64`:

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

### Versions

| | |
|---|---|
| Node | `3.1.1` (`version` in `package.json`) |
| Xray core | `v26.8.15` from `Jolymmiles/Xray-core` (`XRAY_CORE_VERSION` / `UPSTREAM_REPO` in [Dockerfile](./Dockerfile)) |

### Bundled core vs. the runtime core loader

Since upstream `3.1.0` the node ships a `CoreLoaderService`: when the config pushed by the panel
contains a `core` section (`url` + `sha256`), the binary is downloaded at startup into
`/usr/local/bin/xray-custom` and `/usr/local/bin/rw-core` is repointed at it — **the core baked into
this image is then not used**. With no `core` section the service rolls the symlink back to
`/usr/local/bin/xray`, which is the Jolymmiles build from this image.

So keep the panel's `core` section empty to run the bundled smux core, or point it at a smux build
yourself if you prefer managing the core from the panel (note that the loader expects a raw binary,
not the `.zip` published in the Xray-core releases, and gives the download 15 seconds).

### Releasing

Publishing is driven entirely by GitHub Releases:

1. Bump `version` in `package.json` (and `XRAY_CORE_VERSION` / `UPSTREAM_REPO` in `Dockerfile` when the core changes).
2. Create a release on GitHub with the tag of that version (e.g. `v3.1.1`) and keep **Set as the latest release** checked.
   When only the core changes and the node version stays put, add a suffix to the tag (e.g. `v3.1.1-1`).
3. [`.github/workflows/release.yml`](./.github/workflows/release.yml) then:
    - builds `linux/amd64` and `linux/arm64` on native runners and pushes them as a single manifest to
      `ghcr.io/legiz-ru/remnanode-smux:<tag>` (plus `:latest` for non-prereleases);
    - appends a build-info block to the release description with the node version, the Xray core version
      linked to its release page, and the pull commands.

The same workflow can be re-run manually via **Actions → Build & Publish Docker Image → Run workflow**
with an existing tag as input.

### Credits

Based on [remnawave/node](https://github.com/remnawave/node) — check its
[open issues](https://github.com/remnawave/panel/issues) to help the upstream project.
