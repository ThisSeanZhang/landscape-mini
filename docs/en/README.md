# Landscape Mini

[![Latest Release](https://img.shields.io/github/v/release/Cloud370/landscape-mini)](https://github.com/Cloud370/landscape-mini/releases/latest)

English | [中文](../zh/README.md) | [Contributing](../../CONTRIBUTING.md) | [Download Latest Image](https://github.com/Cloud370/landscape-mini/releases/latest)

Landscape Mini is a minimal x86 image builder for Landscape Router. It supports **Debian Trixie** and **Alpine Linux**, can produce `img` / `vmdk` / `ova`, and supports both BIOS and UEFI boot.

Upstream project: [Landscape Router](https://github.com/ThisSeanZhang/landscape)

## Read by Goal

| Your goal | Go to |
|---|---|
| Download a ready-made image | [Release page](https://github.com/Cloud370/landscape-mini/releases/latest) |
| Customize network / passwords / version / output formats | [Custom Build Guide](./custom-build.md) |
| Import / install on PVE | [PVE Installation Guide](./pve-install.md) |
| Build / test / debug locally | Continue reading this page |
| 中文文档 | [docs/zh/README.md](../zh/README.md) |

## Recommended Paths

- Just want to get it running: Release → [PVE Installation Guide](./pve-install.md)
- Need to change network, passwords, or version: [Custom Build Guide](./custom-build.md) → [PVE Installation Guide](./pve-install.md)
- Want to modify this repository: continue with "Local Development" below, then read [CONTRIBUTING.md](../../CONTRIBUTING.md)

## Features

- Supports both Debian and Alpine as base systems
- Build identity is explicitly defined as `base_system + include_docker + output_formats`
- Output formats include `img`, `vmdk`, and `ova`
- Supports both BIOS and UEFI boot
- Fork users can run custom builds directly on GitHub
- GitHub Actions is already set up for build, test, and release

## Local Development

If you already know you want to build locally, debug, or validate unpushed changes, start here.

### Local Build

Local configuration is layered with this precedence:

`build.env < build.env.<profile> < build.env.local < explicit environment variables`

Recommended usage:

- `build.env`: repository defaults, kept tracked
- `build.env.local`: private machine-specific overrides, suitable for passwords, LAN/DHCP, or local test toggles
- `build.env.<profile>`: scenario-specific profiles such as `lab` or `pve`
- explicit environment variables: one-off overrides, for example `LANDSCAPE_ADMIN_USER=bar make build`

```bash
# Install build dependencies (first time only)
make deps

# Default combination: debian + no-docker + img
make build

# Use a profile: build.env.lab
BUILD_ENV_PROFILE=lab make build

# Local private overrides: build.env.local
make build

# Explicit overrides still have the highest precedence
LANDSCAPE_ADMIN_USER=admin RUN_TEST=readiness make build

# Alpine raw image
make build BASE_SYSTEM=alpine

# Debian + Docker + img,ova
make build INCLUDE_DOCKER=true OUTPUT_FORMATS=img,ova
```

Common local customization inputs include:

- `LANDSCAPE_ADMIN_USER` / `LANDSCAPE_ADMIN_PASS`
- `LANDSCAPE_LAN_SERVER_IP` / `LANDSCAPE_LAN_RANGE_START` / `LANDSCAPE_LAN_RANGE_END` / `LANDSCAPE_LAN_NETMASK`
- `RUN_TEST`

### Local Test

```bash
# Automated readiness checks (non-interactive)
make deps-test
make test

# Dataplane tests only apply to include_docker=false raw images
make test-dataplane

# You can also run validation automatically after a build
RUN_TEST=readiness make build
RUN_TEST=readiness,dataplane make build

# When INCLUDE_DOCKER=true, requested dataplane is skipped explicitly and recorded as a skip marker
INCLUDE_DOCKER=true RUN_TEST=readiness,dataplane make build

# You can also point tests at any raw image directly
./tests/test-readiness.sh output/landscape-mini-x86-alpine.img
./tests/test-dataplane.sh output/landscape-mini-x86-debian.img

# Interactive boot (serial console)
make test-serial
```

## Deployment

### Physical Machine / USB Drive

```bash
dd if=output/landscape-mini-x86-debian.img of=/dev/sdX bs=4M status=progress
```

### Proxmox VE (PVE)

Start with:

- [PVE Installation Guide](./pve-install.md)

### Cloud Server (dd Script)

Use the [reinstall](https://github.com/bin456789/reinstall) script to write the custom image to a cloud server:

```bash
bash <(curl -sL https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh) \
    dd --img='https://github.com/Cloud370/landscape-mini/releases/latest/download/landscape-mini-x86-debian.img.gz'
```

> The root partition automatically expands on first boot to fill the whole disk.

## Default Credentials

| Scenario | Username | Password |
|------|------|------|
| SSH / system login | `root` | `landscape` |
| SSH / system login | `ld` | `landscape` |
| Web UI | `root` | `root` |

> `custom-build.yml` can override Linux / Web UI credentials through workflow inputs or GitHub Secrets. Plaintext inputs are fine for temporary personal use; if security matters, prefer `CUSTOM_ROOT_PASSWORD`, `CUSTOM_API_USERNAME`, and `CUSTOM_API_PASSWORD`.

## Build Configuration

Avoid using tracked `build.env` as the day-to-day customization entry point. Prefer:

- `build.env.local`
- `build.env.<profile>`
- explicit environment variables
- GitHub Actions `Custom Build`

| Variable | Default | Description |
|------|--------|------|
| `BASE_SYSTEM` | `debian` | Base system: `debian` / `alpine` |
| `INCLUDE_DOCKER` | `false` | Include Docker: `true` / `false` |
| `OUTPUT_FORMATS` | `img` | Output formats: `img`, `vmdk`, `ova` (comma-separated) |
| `RUN_TEST` | _(empty)_ | Local test selection: empty / `none`, `readiness`, `readiness,dataplane` |
| `LANDSCAPE_ADMIN_USER` | `root` | Web admin username |
| `LANDSCAPE_ADMIN_PASS` | `root` | Web admin password |
| `LANDSCAPE_LAN_SERVER_IP` | _(empty)_ | LAN gateway / DHCP service IP |
| `LANDSCAPE_LAN_RANGE_START` | _(empty)_ | LAN DHCP range start |
| `LANDSCAPE_LAN_RANGE_END` | _(empty)_ | LAN DHCP range end |
| `LANDSCAPE_LAN_NETMASK` | _(empty)_ | LAN subnet prefix length, for example `24` |
| `APT_MIRROR` | _(auto probe)_ | Explicit Debian package mirror override; if empty, candidates are auto-detected |
| `ALPINE_MIRROR` | _(auto probe)_ | Explicit Alpine package mirror override; if empty, candidates are auto-detected |
| `DOCKER_APT_MIRROR` | _(auto probe)_ | Explicit Debian Docker APT repository override; if empty, candidates are auto-detected |
| `DOCKER_APT_GPG_URL` | _(auto probe)_ | Explicit Debian Docker APT GPG key URL override; if empty, candidates are auto-detected |
| `LANDSCAPE_VERSION` | `v0.18.2` | Upstream Landscape version |
| `LANDSCAPE_REPO` | `https://github.com/ThisSeanZhang/landscape` | Upstream Landscape release repository |
| `IMAGE_SIZE_MB` | `2048` | Initial image size (shrunk automatically later) |
| `ROOT_PASSWORD` | `landscape` | Login password for `root` / `ld` |
| `TIMEZONE` | `Asia/Shanghai` | Time zone |
| `LOCALE` | `C.UTF-8` | Default system locale |
| `EXTRA_LOCALES` | `en_US.UTF-8` | Additional Debian UTF-8 locales to generate |

### Custom Build (GitHub Actions)

The repository provides `custom-build.yml` as an explicit-tuple build entry point for fork users. It supports:

- `base_system`: `debian` / `alpine`
- `include_docker`: `true` / `false`
- `output_formats` (use `ova` as the canonical OVA output format name)
- `landscape_version`
- `lan_server_ip` / `lan_range_start` / `lan_range_end` / `lan_netmask`
- `root_password`
- `api_username` / `api_password`
- `run_test`

Current precedence: **direct inputs > secrets > defaults**.

The workflow validates inputs first, catching invalid `output_formats`, `run_test`, and basic network input errors before the build starts.

The unified test contract is:

- empty / `none`: build only
- `readiness`
- `readiness,dataplane`

When `include_docker=true`, requested dataplane is explicitly skipped with a reason in the logs.

The workflow writes the following identity fields into `build-metadata.txt`:

- `base_system`
- `include_docker`
- `output_formats`
- `run_test`
- `produced_files`
- `artifact_id`
- `release_channel`

The effective network topology is shipped inside the artifact as `effective-landscape_init.toml` for `test.yml`, fixed-release publishing, and tag release rebuild validation.

Successful Custom Build runs also publish to the fixed tag `custom-build-latest` in the fork.
It is a moving pointer to the latest successful Custom Build rather than a per-tuple permanent download slot; any later successful build overwrites it.

- Release page: `https://github.com/<owner>/landscape-mini/releases/tag/custom-build-latest`
- Direct download base: `https://github.com/<owner>/landscape-mini/releases/download/custom-build-latest/<asset>`

If you need immutable per-build outputs, use the Artifacts from that workflow run or record its `run_id` / `artifact_id`.

## Automated Testing

### Readiness Checks

`make test` or `./tests/test-readiness.sh <image.img>` runs the shared router readiness contract:

1. Copy the image to a temporary file to protect the build artifact
2. Start QEMU in the background and auto-detect KVM
3. Wait for SSH, API listener, API login, and layout detection
4. Verify `eth0` / `eth1` and core services reach the running state
5. When `include_docker=true`, additionally verify Docker is available
6. Emit readiness / service / diagnostics snapshots and clean up QEMU

### Dataplane Tests

`make test-dataplane` or `./tests/test-dataplane.sh <image.img>` validates real client-visible dataplane behavior with a two-VM topology:

```text
Router VM (eth0=WAN/SLIRP, eth1=LAN/mcast) ←→ Client VM (CirrOS, eth0=mcast)
```

Coverage includes DHCP lease assignment, lease visibility in the Router API, and LAN connectivity between Router and Client.

> Dataplane scheduling is based on `include_docker=false`, not legacy variant names.

## CI/CD

- **CI**: Manual runs are always available. Automatic `push main` / `PR -> main` runs trigger only when shell or CI execution logic changes.
- **Fork protection**: automatic events in forks are skipped by default; manual dispatch remains available.
- **Automatic CI validation surface**: only `debian + include_docker=false`, requesting raw `img` only.
- **Readiness / dataplane coverage**: automatic CI runs `readiness,dataplane` for `include_docker=false`.
- **Artifact contract**: every image artifact includes raw `.img`, `build-metadata.txt`, and `effective-landscape_init.toml`; automatic CI no longer exports `.vmdk` / `.ova`.
- **Custom Build**: `custom-build.yml` lets fork users build explicit tuples, validates inputs early, and provides clearer download / import guidance.
- **Manual Retest**: `test.yml` retests the Debian default public tuple by `run_id` or `artifact_id`, with SSH / API credentials passed in again.
- **Release**: when a `v*` tag is pushed, `release.yml` rebuilds Debian Docker / non-Docker artifacts from that tagged commit instead of promoting CI artifacts, and publishes the default public surface: `.img.gz` + `.ova`.
- **Alpine**: Alpine is no longer part of the default public release surface; use `Custom Build` when you need it.

## License

This project is released under **GPL-3.0**, consistent with upstream [Landscape Router](https://github.com/ThisSeanZhang/landscape).
