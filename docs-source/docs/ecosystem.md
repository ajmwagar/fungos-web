---
title: Component ownership and interfaces
source_commit: 6bb83c9bb3b650483e0dbcb9c6a14b37906671b9
watches:
  - README.md
  - base/interfaces.md
---
# Component ownership and interfaces

fungOS is a Debian-based operating environment that composes independently useful tools. Not every node runs every application. Shared contracts should be small, versioned and owned once; a discovery observation or shared schema is not permission to read another application's internal state.

## Ownership map

| Component | Owns | Does not own |
| --- | --- | --- |
| fungOS | Runtime profiles, generic rootfs assembly and provisioning seams | Every application's implementation or license |
| Mycelium | Machine identity, discovery, topology, access and signed distribution | Application transport or inference kernels |
| Genesis | Bootstrap, disk/image provisioning and first identity delivery | Ongoing application message routing |
| [Unibus](https://fungos.dev/projects/unibus.html) | Authorized application envelopes, routing and transport | Domain effects or machine lifecycle |
| Canvas | Human/agent composition, surfaces and application-owned controls | Host provisioning or generic transport |
| Bifrost | Inference-facing interfaces and routing | Machine identity or generic service supervision |
| [UMIE](https://fungos.dev/projects/umie.html) | Modular inference execution and model recipes | Fleet authority or arbitrary model compatibility |
| Yggdrasil | Compute placement/execution integration | Issuing host identity |
| Shroud | Workload packaging and execution substrate | Bare-metal provisioning authority |
| IPFS/storage providers | Content-addressed bytes and caching | Application permission policy |
| [DCP](https://fungos.dev/projects/dcp.html) | Bounded state-derived decision contract | Provider internals or execution authority |
| [Holodeck](https://holodeck.fpl.dev/) | XR room, interaction and bounded Lua room interfaces | CAD/CAM, audio or feed-owner internals |

This table describes interfaces and ownership, not a claim that each integration is installed, public or production-qualified.

## Select a deployment

- **Base node:** generic Debian prerequisites, first-contact and update verification.
- **Headless edge:** service and message runtime without a local desktop.
- **Display edge:** Canvas runtime prerequisites, plus separately installed native applications.
- **Compute node:** explicitly placed inference/accelerator services; resource observations inform eligibility, not trust.
- **Existing workstation:** independently installed services retain the native OS and security model.

Choose [profiles](https://fungos.dev/docs/profiles.html) from actual requirements. A GPU does not automatically grant a compute role; a reachable router does not authorize a MIDI output; a model CID does not grant access to private content.

## Interfaces worth preserving

1. First contact delivers a protected machine-specific claim through a provisioner-owned adapter.
2. Update verification consumes artifact, manifest and signature paths without owning installation.
3. Application transport consumes narrow domain contracts without interpreting their effects.
4. DCP execution consumes exact catalog/state revisions and provider-defined arguments.
5. Inference consumers use the engine's declared model recipes and interfaces rather than backend internals.
6. XR programs consume the versioned room ABI, not credentials or private applet state.

## Operational status

The public fungOS builder and runtime interfaces are available. Generic outputs are rootfs tarballs; there is no public installer ISO in this slice. Persistent amd64 QEMU edge behavior and isolated visual rollback have documented qualification. Board boot integration and public package-origin deployment remain separate work.

Public owner sources: fungOS `README.md`, `base/interfaces.md`, `docs/edge-dogfood.md`. The broader ownership map is reviewed against the original Mycelium `fungOS/ecosystem.md` snapshot and Unibus/DCP contracts; freshness tracking here watches the public fungOS source, not every independent project.
