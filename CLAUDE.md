# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

Landscape Mini builds minimal x86 images for Landscape Router.

- Base systems: Debian Trixie / Alpine Linux
- Boot: BIOS + UEFI
- Build identity model: `base_system + include_docker + output_formats`
- Upstream project: https://github.com/ThisSeanZhang/landscape

## Common Commands

```bash
make deps
make deps-test
make build
make build BASE_SYSTEM=alpine
make build INCLUDE_DOCKER=true OUTPUT_FORMATS=img,pve-ova
make test
make test-dataplane
make test-serial
make ssh
```

## Defaults and important inputs

- Default upstream version comes from `build.env` (`LANDSCAPE_VERSION`, currently `v0.18.2`)
- Common build env overrides:
  - `BASE_SYSTEM`
  - `INCLUDE_DOCKER`
  - `OUTPUT_FORMATS`
  - `ROOT_PASSWORD`
  - `LANDSCAPE_ADMIN_USER`
  - `LANDSCAPE_ADMIN_PASS`
  - `EFFECTIVE_CONFIG_PATH`
  - `APT_MIRROR`
  - `ALPINE_MIRROR`

## Build and test contract

- CI and Custom Build both use `.github/workflows/_build-and-validate.yml`
- Each image artifact must include:
  - raw `.img`
  - `build-metadata.txt`
  - `effective-landscape_init.toml`
- Tests should use effective topology config and build metadata
- Dataplane scheduling rule:
  - `include_docker=false` → run dataplane
  - `include_docker=true` → readiness only

## CI/CD summary

### CI

`ci.yml` builds 4 tuples:

- `debian + false`
- `debian + true`
- `alpine + false`
- `alpine + true`

### Custom Build

`custom-build.yml` is the fork-friendly manual entry point.

Supports:

- `base_system`
- `include_docker`
- `output_formats`
- `landscape_version`
- LAN / DHCP inputs
- Linux password
- Web admin username / password

### Retest

`test.yml` retests existing CI artifacts by `run_id` or `artifact_id`.

### Release

`release.yml` does promotion, not rebuild.
