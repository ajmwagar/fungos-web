---
title: Signed updates, repair and rollback
source_commit: 6bb83c9bb3b650483e0dbcb9c6a14b37906671b9
watches:
  - base/interfaces.md
  - docs/edge-dogfood.md
  - docs/visual-rollback.md
  - packages
---
# Signed updates, repair and rollback

fungOS supplies an update-verification seam; Mycelium owns the qualified native activation path. A signed APT publisher is also available for explicitly selected package ownership. **Do not give APT and native activation competing ownership of one executable.**

## Verification is separate from installation

`fungos-verify-update ARTIFACT MANIFEST SIGNATURE` invokes exactly one installed verifier. Exit zero authorizes staging; the runner does not install bytes. The verifier owns signature format, trusted keys, expiry checks, rollback protection and manifest schema.

Generic images carry no release signing private key. A source URL, announced version or hash alone is not authority to activate an artifact.

## Native update workflow

1. A build provider produces immutable executable artifacts.
2. The deployment's release authority signs the release metadata.
3. The update owner verifies metadata and bytes, evaluates placement/channel policy and stages the candidate.
4. Native systemd/launchd bindings restart the application and evaluate owner-provided readiness.
5. A healthy candidate becomes current; a failed candidate follows the owner's recovery path.
6. Preserve the activation record, errors and prior verified artifact for investigation.

A process being active is weaker evidence than application readiness. For a compositor, include renderer/output health and actual framebuffer evidence.

## Routine operator checks

On an enrolled node with the configured Mycelium integration:

```sh
mycelium software reconcile --dry-run --json
mycelium update policy status --json
systemctl status mycelium-update.timer
journalctl -u mycelium-update.service
```

Check signer, channel, architecture, artifact availability, activation status and native owner readiness. Keep age, rollout and retry gates; do not delete an activation checkpoint to make a failed update look successful.

## Qualified evidence

| Qualification | Established | Not established |
| --- | --- | --- |
| Persistent QEMU dogfood, 8 October 2026 | Same-version Unibus repair; unchanged identity and Canvas workspace through reboot | A new release upgrade |
| Offline visual rollback fixture | Signed failing Canvas and Unibus candidates roll back; saved state and enrolled identity survive reboot | Fresh enrollment, production automatic rollout or multi-package atomicity |
| Signed APT qualification | Debian client verification in the owner's isolated test | A deployed public APT origin or automatic fleet policy |

The visual fixture uses a disposable offline copy and disposable signer. Its recorded checks include recovered executable/link bytes, application readiness, cleared pending checkpoints, unchanged state and a clean reboot. It must never be run against a production device.

## Recovery runbook

1. Keep failed logs and the exact candidate/release identity.
2. Determine whether the failure is download, signature, policy, activation or application readiness.
3. Restore through the activation owner's supported recovery path, preserving the prior verified artifact.
4. Verify application output and retained state, then perform a controlled reboot.
5. Correct the cause before retrying; do not weaken trust checks or re-enroll healthy machine identity.

## Current limits

The public origin is not deployed. Compositor production updates remain gated on application-owned renderer/output health. The recorded tests do not establish whole-set atomic updates, application-data migration safety, Pi support, encrypted-root unlock or TV media delivery.

Sources: public fungOS `base/interfaces.md`, `docs/edge-dogfood.md`, `docs/visual-rollback.md`, `packages/README.md`, `packages/QUALIFICATION.md`.
