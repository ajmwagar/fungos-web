---
title: Build and inspect fungOS
source_commit: 6bb83c9bb3b650483e0dbcb9c6a14b37906671b9
watches:
  - base
---
# Build and inspect fungOS

The public fungOS source builds capability-derived Debian root filesystems for `amd64` and `arm64`. The output is a **rootfs tarball**, not a bootable disk, SD-card image or installer ISO. Kernel, firmware, disk assembly, enrollment and signed first-party applications remain separate steps.

This guide follows the public [fungOS source](https://github.com/ajmwagar/fungos) at the reviewed revision above. First-party code and documentation are MIT OR Apache-2.0; packaged Linux, Debian and optional applications retain their own licenses.

## Prerequisites

Use a Debian-compatible build host with `mmdebstrap`, `debian-archive-keyring`, `tar`, `sha256sum` and `shellcheck`. Cross-building arm64 additionally needs `qemu-user-static` and `binfmt-support`. The build needs root or a working rootless mmdebstrap mode, outbound HTTPS and enough free space for one root filesystem.

Use committed `base/config/release.env` inputs. Changing the locked snapshot changes the artifact; review that change before calling it a release.

## Build runbook

From the repository root:

```sh
./base/scripts/check.sh
sudo ./base/scripts/build.sh amd64 base
sudo ./base/scripts/inspect.sh base/out/fungos-base-amd64.tar
```

1. Run the inexpensive declaration and shell checks first.
2. Select a supported architecture and [profile](https://fungos.dev/docs/profiles.html).
3. Build on a suitable Linux host.
4. Retain the tarball, manifest, resolved package list and SHA-256 file from `base/out/`.
5. Inspect the artifact before disk assembly or provisioning.

The builder refuses unsupported architectures and unknown or duplicate capabilities. `OUT_DIR` and `WORK_DIR` allow independent CI scratch directories. Do not share a work directory between concurrent builds.

## Verify reproducibility

Build twice from clean work directories with the same committed inputs, then compare both artifact digests and package lists:

```sh
sha256sum out-a/fungos-base-amd64.tar out-b/fungos-base-amd64.tar
diff -u out-a/fungos-base-amd64.packages out-b/fungos-base-amd64.packages
```

Matching inputs should produce matching digests. If they differ, check dependency snapshots, undeclared inputs and archive normalization.

## Provisioning interfaces

| Interface | Contract |
| --- | --- |
| `/usr/lib/fungos/first-contact.d/` | Exactly one executable adapter; receives the first-contact environment file path |
| `/etc/fungos/first-contact.env` | Provisioner-owned `KEY=VALUE` records; adapter validates them |
| `/usr/lib/fungos/update-verifiers.d/` | Exactly one executable verifier; receives artifact, manifest and signature paths |
| `fungos-verify-update` | Exit zero authorizes staging; it does not install the artifact |

The generic image does not embed tokens, endpoints, enrollment certificates or a Mycelium executable. First-contact adapters own transport, retries and durable enrollment state. Update verifiers own trusted keys, signature format, expiry and rollback protection.

An absent or ambiguous adapter fails visibly. Never let filesystem ordering select a security policy.

## Recovery

Inspect `journalctl -u fungos-first-contact`, networkd state, adapter ownership and executable permissions. Remove stale adapters rather than weakening the single-adapter rule. Provision a public-key SSH recovery path; root password login is disabled in the generic image.

See [installation](https://fungos.dev/docs/install.html) for assembling a bootable target and [signed updates](https://fungos.dev/docs/updates.html) for activation and recovery. Owner references: `base/README.md`, `base/RUNBOOK.md`, `base/interfaces.md` and `base/scripts/`.
