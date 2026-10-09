---
title: Raspberry Pi support and qualification
source_commit: 6bb83c9bb3b650483e0dbcb9c6a14b37906671b9
watches:
  - README.md
  - base/scripts/build.sh
  - base/profiles
---
# Raspberry Pi support and qualification

The public fungOS builder supports an `arm64` Debian rootfs. **That is not a ready-to-flash Raspberry Pi image.** Board firmware, kernel, modules, boot partition, disk assembly and hardware qualification are separate steps.

The public repository states that older Pi integration work is being separated from the Mycelium development checkout and is not all migrated. This page intentionally does not turn those experiments into a supported installer.

## Build the generic runtime

Cross-build on a suitable Linux build host, not on the target Pi. An architecture change needs `qemu-user-static` and `binfmt-support` in addition to the standard builder prerequisites.

```sh
./base/scripts/check.sh
sudo ./base/scripts/build.sh arm64 headless-edge
sudo ./base/scripts/inspect.sh base/out/fungos-headless-edge-arm64.tar
```

Use `edge` only when you also intend to supply and qualify the board's display stack. A headless profile excludes Canvas display prerequisites but still requires separately installed application executables and secure provisioning.

## Board integration checklist

1. Choose the exact board revision and a compatible firmware/kernel/module set.
2. Assemble a fresh disk image from the inspected rootfs; retain serial or public-key SSH recovery.
3. Use a machine-specific first-contact adapter and protected enrollment claim.
4. Verify network, persistent disk, USB devices and thermals on the actual board.
5. Test restart and full reboot without enrollment media; compare identity and application state.
6. If using audio or display, inspect the real peripheral output, not just unit status.
7. Record the precise board, boot inputs, application versions and failed cases before calling it qualified.

This is a qualification checklist, not an executable flashing script. Do not write the rootfs tarball directly to an SD card or overwrite an existing device to test it.

## Boundaries

The amd64 QEMU evidence does not establish Pi hardware support. A CPU architecture match also does not establish GPU, audio or USB compatibility. Existing Raspberry Pi OS nodes can use independently installed services without reimaging; doing so is different from installing fungOS.

Sources: public fungOS `README.md`, `base/scripts/`, `base/profiles/`. Follow [build prerequisites](https://fungos.dev/docs/base.html), [profile selection](https://fungos.dev/docs/profiles.html) and [first-contact contracts](https://fungos.dev/docs/install.html).
