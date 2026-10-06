# Trivy (snap)

A Snap package of [Trivy](https://trivy.dev/) (GitHub:
[aquasecurity/trivy](https://github.com/aquasecurity/trivy)) — a comprehensive
security scanner for vulnerabilities, misconfigurations, secrets and SBOMs,
available from the Snap Store here:

https://snapcraft.io/trivy-bp

> **Community / unofficial.** This is an unofficial, community-maintained snap.
> The snap `name` is `trivy-bp`; the `title` is `Trivy` so the software is
> recognised by its real name.

## What's in the snap

The snap installs one binary from upstream, with a suggested alias:

| Binary      | Alias   | Purpose                                              |
| ----------- | ------- | ---------------------------------------------------- |
| `bin/trivy` | `trivy` | Scan images, filesystems, repos, SBOMs, Kubernetes   |

To enable that alias, after installing:

  ```sh
  sudo snap alias trivy-bp.trivy trivy
  ```

The binary is built statically (`CGO_ENABLED=0`) from the source of a
[pinned upstream release](https://github.com/aquasecurity/trivy/releases).
The pinned tag lives in `snap/snapcraft.yaml`; Renovate bumps it via a pull
request whenever aquasecurity/trivy publishes a new release, so upgrades are
reviewed rather than picked up silently on the next build.

## Installing

On a snap that has been promoted to `stable`:

```sh
snap install trivy-bp
```

From the `edge` track:

```sh
snap install trivy-bp --edge
```

### Interfaces / filesystem access

The snap runs under **strict confinement** and declares the following plugs on
the app:

- `home` — read/write access to `$HOME`. This is **auto-connected** on
  install, and is what lets you scan files and repositories under your home
  directory.
- `removable-media` — access to mounted removable media (`/media`, `/mnt`).
  This is **not** auto-connected; enable it explicitly if you need it:

  ```sh
  sudo snap connect trivy-bp:removable-media
  ```

- `network` — outbound network access. This is **auto-connected**, and is
  what lets Trivy download its vulnerability and Java databases and checks
  bundles, and pull images from remote registries.
- `network-bind` — listen on network sockets. This is **auto-connected**, and
  is needed for `trivy server`.
- `docker` — access to `/var/run/docker.sock`, so `trivy image` can scan
  images held by the local Docker daemon. This grants privileged access to the
  system and is **not** auto-connected; enable it explicitly if you need it:

  ```sh
  sudo snap connect trivy-bp:docker
  ```

  Without it, `trivy image` still pulls images directly from registries.

The vulnerability DB cache lives in `~/snap/trivy-bp/common/.cache/trivy`
(`XDG_CACHE_HOME` is set to `$SNAP_USER_COMMON/.cache`), so it is kept across
snap refreshes rather than copied per revision. Override it with
`--cache-dir` if you need it elsewhere.

> **Note on paths.** Strict confinement limits access to paths outside
> `$HOME`, the snap's own mount points, and `/media`/`/mnt`. Paths elsewhere —
> e.g. `trivy rootfs /` or files under `/tmp` — are not readable; you may need
> to copy or mount them somewhere under the accessible paths, or request an
> additional interface.
>
> Hidden configuration under `$HOME`, such as `~/.docker/config.json` and
> `~/.kube/config`, is also not readable under strict confinement. Pass
> registry credentials via environment variables (e.g. `TRIVY_USERNAME` /
> `TRIVY_PASSWORD`), or copy your kubeconfig to a non-hidden path and use
> `--kubeconfig`.

## Building locally

Requires `snapcraft` (install with `sudo snap install snapcraft --classic`):

```sh
snapcraft
```

This builds `trivy-bp_<version>_<arch>.snap` in the project root. You can also
run a local lint:

```sh
snapcraft lint
```

## Continuous integration

[`.github/workflows/snap.yml`](.github/workflows/snap.yml) is CI-only:

- Builds the snap on every push / pull request.
- Lints the built snap with `snapcraft lint` in the same job (a hard gate:
  lint failures fail the build).

This workflow is a **merge gate only** — it never publishes. Building **and**
publishing to `latest/edge` is handled by snapcraft.io, which schedules its
own builds of this snap and pushes them to the `edge` track automatically.
Promoting to a `stable` track remains a manual decision (via the Snap Store
dashboard). This repo is intentionally never tagged — the upstream `trivy` repo
is where the release tags live.

## License

The packaged software (Trivy) is licensed under the Apache License 2.0, as
recorded in `snap/snapcraft.yaml`.
