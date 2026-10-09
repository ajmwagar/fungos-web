---
title: Unibus — transport, grants and integration
source_commit: 98e842156554def8089882d2e1e9917e49253cb8
watches:
  - crates/unibus
  - crates/unibus-router
  - crates/unibus-sdk
  - docs/sdk.md
  - docs/direct-sessions-and-discovery.md
---
# Unibus

Unibus is a **transport protocol**, with Rust libraries, routers and service adapters implementing it. It carries display, audio, presence, sensor and control messages between independently useful applications. It does not own their data models, machine provisioning or execution policy.

This guide describes the reviewed source revision above. Protocol definitions and executable tests remain in the [Unibus repository](https://github.com/FuturePresentLabs/unibus). Repository access may be required. An available adapter is not evidence that a deployment has configured it.

## Architecture and ownership

| Layer | Owner | Responsibility |
| --- | --- | --- |
| L1 transport | Unibus | Envelopes, framing, freshness, routing, grants and scheduling |
| L2 domain contracts | Each application | Message meaning, schemas, state, validation and effects |
| Identity and discovery | Deployment-selected provider | Certificates, trusted peers and candidate endpoints |
| Bulk assets | Storage/content provider | Models, media and content-addressed bytes |

Use one provider-owned contract for each domain. Canvas owns its composition commands; a MIDI edge owns exact device packets; DCP providers own legal actions. The router routes by destination, screen anchor, zone, capability and lifetime; application bodies remain opaque.

## Wire contract

The stream format is a four-byte unsigned big-endian length followed by a JSON envelope and optional binary payload. The length covers the envelope and payload, excluding its own prefix. The decoder requires the declared binary byte count to match the trailing bytes.

The envelope carries schema version, installation and producer identity, destination or screen target, request identity, timestamp, lifetime, binary payload size and a message whose routing name is `message_type`. Schema versions 1 and 2 share framing; golden wire fixtures protect compatibility with the original router.

The current L1 maximum frame is 16 MiB, with a default control freshness window of 30 seconds and bounded future clock skew. These are validation bounds, not recommended media packet sizes. Put bulk model and audio files on the content plane, and send small references or control events on the bus.

Commands, streams and snapshots use separate scheduling classes with bounded peer queues. Delivery policy is contract-specific; do not assume an acknowledged enqueue means an application applied an effect.

## Transport choices

| Path | Use | Security boundary |
| --- | --- | --- |
| Routed TCP | Local router and compatibility deployments | Plaintext listeners are loopback-only by default; remote use needs an explicitly secured path |
| Routed QUIC/TLS | Encrypted router connections | Configure certificate, key and authenticated server name |
| Direct QUIC/mTLS | Authorized peer-to-peer data | Verify signed session claims and both peer certificate bindings |
| Optional discovery adapters | Static configuration, DNS or Mycelium | Reachability does not grant authority |

Direct-session claims bind both producer identities, peer keys, exact allowed kinds, payload bounds, endpoints and expiry. A failed direct connection may use an explicitly supplied routed fallback; it must not weaken authentication. Mycelium can supply discovery or certificates without carrying or interpreting Unibus frames.

## Grants and privacy

Node grants cover zones, namespaced capabilities, screen anchors, site and role. Schema-v2 publishers explicitly declare permitted kinds and publishing zones, anchors and producers. Authentication ends at registration; downstream services must still authorize domain effects.

Keep credentials in host-managed files or environment, never in catalogs, scripts, asset manifests or discovery advertisements. Grant a MIDI source only its intended destination and kinds. Separate permission to see a Canvas surface from permission to interact with it. A valid DCP catalog is not an execution grant.

An authorized recipient can retain received bytes. Transport grants do not promise screenshot prevention, unsaveable media or retroactive erasure. Private asset distribution also needs the storage owner's access policy.

## Adapters and interfaces

| Implementation | Contract / scope | Application responsibility |
| --- | --- | --- |
| MQTT bridge | Exact topics and binary payloads over `mqtt.publish.v1` | Broker sessions, retained state and MQTT acknowledgements |
| MIDI bridge | Ordered exact packets over `midi.packet.v1` | Explicit input/output device selection and performance validation |
| ADB bridge | Portable remote device attachment and operations | USB authorization, device access and credential handoff |
| MCP edge | Leased provider tools and invocation routing | Tool semantics and mutation authority |
| DCP edge | Canonical revision-bound catalogs and execution receipts | Finite actions, state checks and execution |
| Embedded SDK | Rust session; Lua/Luau and C ABI bindings | VM lifecycle, update budget, resynchronization and sandbox policy |

MCP and DCP are parallel bindings. Unibus does not turn arbitrary tools into bounded DCP actions. The DCP client edge revalidates live provider leases and exact catalog/state revisions before forwarding execution.

## Embedded SDK limits

`publish` queues an object body and returns a request ID. Consume replies to determine delivery or authorization. `poll` is nonblocking; `status` reports connection state and dropped events. Reconnection is explicit and commands are not replayed automatically.

Both queues are bounded to 128 entries; control bodies are limited to 64 KiB. Full outbound queues error. Dropped inbound events increment a counter; stateful clients must resynchronize. Network work stays on a Rust worker, while VM callbacks stay on the host thread. Do not connect or close inside a rendering frame.

Lua hosts select exactly one mlua backend: Luau by default, LuaJIT for compatible VR hosts, or Lua 5.4. Untrusted scripts should receive a host-selected session through `attach`, rather than permission to choose credentials or connect freely. Router grants remain authoritative.

## First integration runbook

1. Choose one narrow, versioned domain message contract and an explicitly addressed test recipient.
2. Check and test only the required crates. Linux MIDI/audio builds need ALSA development libraries; portable ADB builds do not.
3. Configure a local router, trusted registration credentials and minimum sender/receiver grants.
4. Start the recipient before the source; confirm registration and route selection.
5. Send one deterministic event. Verify the reply and the recipient's actual state separately.
6. Test expiry, denied publishing, disconnect and receiver restart. Stateful sources must send a fresh snapshot after gaps.
7. Add encrypted remote routing or direct sessions only after the local contract works.

```sh
cargo check --locked -p unibus-adb
cargo test --locked -p unibus-adb

# Embedded host contracts:
cargo check -p unibus-sdk -p unibus-lua -p unibus-ffi
cargo test -p unibus-sdk -p unibus-lua -p unibus-ffi
```

For an already configured local MCP edge, `unibus-mcp setup-clients` previews client configuration; `--write` applies it. This configures the client endpoint, not provider credentials or grants.

## Source references

- `README.md`: workspace, adapters, routing and grants.
- `crates/unibus/src/envelope.rs` and `framing.rs`: actual wire bounds and codec.
- `docs/sdk.md`: embedded API, ownership and overflow behavior.
- `docs/direct-sessions-and-discovery.md`: signed direct sessions and provider boundaries.
- `docs/midi-bridge-runbook.md`, `docs/mqtt-bridge-runbook.md`, `docs/adb-portable-quickstart.md`: operational recipes.

See the [DCP integration guide](https://fungos.dev/projects/dcp.html) for bounded decisions and the [fungOS component map](https://fungos.dev/docs/ecosystem.html) for ownership across the environment.
