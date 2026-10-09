---
title: Profiles and deployment boundaries
source_commit: 6bb83c9bb3b650483e0dbcb9c6a14b37906671b9
watches:
  - base/profiles
  - base/capabilities
  - README.md
---
# Profiles and deployment boundaries

Profiles derive Debian package sets from named capability declarations. They describe runtime prerequisites, not an application bundle, a placement grant or a complete bootable image.

## Profile matrix

| Profile | Declared capabilities | Intended use | Does not include |
| --- | --- | --- | --- |
| `base` | Core, networkd, SSH, first contact, signed-update verification | Small provisionable node | Application executables, identity or board firmware |
| `edge` | Base plus edge runtime and Canvas display dependencies | Native visual edge | Automatically authorized devices or a preconfigured production seat |
| `headless-edge` | Base plus edge runtime | Messaging/services without a local desktop | Canvas display runtime |
| `cloud` | Base plus cloud runtime | Workload substrate prerequisites | Edge display packages or a required GPU |

Both `amd64` and `arm64` are declared targets. Architecture support for a rootfs does not prove that a particular board boots or that an application supports its accelerator.

## Selecting and inspecting

```sh
./base/scripts/check.sh
sudo ./base/scripts/build.sh amd64 edge
sudo ./base/scripts/inspect.sh base/out/fungos-edge-amd64.tar
```

Inspect the resolved package list and manifest before provisioning. Package selections are defined in `base/profiles/*.capabilities` and `base/capabilities/*.packages`.

## Runtime dependencies versus applications

Canvas, Unibus, Mycelium and UMIE remain independent owners. A profile can supply their runtime libraries; their signed executables, trust configuration, data and service policy arrive separately. GPU observation may inform eligibility but cannot grant a workload role or permission to execute.

Keep one activation owner per executable. APT and native package activation must not race to manage the same application. Choose the owner explicitly and retain its recovery path.

## Security boundaries

- First-contact and update-verifier adapters use an exact single-adapter contract.
- Enrollment claims are private, short-lived and machine-specific; generic images must not carry them.
- Discovery establishes facts about available resources, not authority to place workloads or control peripherals.
- A role or signed placement policy is separate from the machine's hardware capabilities.
- Generated distributions contain separately licensed software; the complete image is not uniformly MIT/Apache.

## Tested configurations

The public owner documents persistent amd64 QEMU edge operation and isolated visual rollback. The generic builder itself is not an installer. Pi, cloud disk assembly, encrypted-root unlock, whole-set atomic updates and accelerator-specific model qualification have independent evidence requirements.

Follow [installation](https://fungos.dev/docs/install.html), [Pi status](https://fungos.dev/docs/pi.html) or [update qualification](https://fungos.dev/docs/updates.html). Reviewed sources: public fungOS `README.md`, `base/profiles/`, `base/capabilities/`, `THIRD-PARTY.md`.
