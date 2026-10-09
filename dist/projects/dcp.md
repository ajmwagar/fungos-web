---
title: DCP — bounded decisions and provider integration
source_commit: 398408c33d4a5ee3e0eecbe612e9e91cb0896fca
watches:
  - spec/v0.1
  - public/schemas/v0.1
  - conformance/v0.1
---
# Decision Catalog Protocol

DCP publishes the finite decisions a system can legally make against its current state. A client selects an advertised action and schema-valid arguments; the provider validates authority and current revisions before applying an effect.

**Version 0.1 is a draft.** This ecosystem guide is explanatory, not normative. The [DCP reference site](https://dcp.fpl.dev/) and [normative specification](https://github.com/FuturePresentLabs/dcp/blob/main/spec/v0.1/README.md) own the protocol. Reviewed source: `spec/v0.1/README.md` and the JSON Schema 2020-12 contracts at the revision above.

## Why catalogs are state-derived

A machine's legal actions change as its state changes. A stock editor can offer only material and dimensions that fit the current blank; a room can offer only module placements legal in its current floorplan. The provider computes these choices. The decision system does not invent internal function names, shell commands or object handles.

The flat `actions` collection is authoritative. A decision tree may present those same action IDs, but it does not create additional authority.

## Core HTTP surface

| Endpoint / header | Purpose |
| --- | --- |
| `GET /.well-known/dcp` | Provider identity, exact supported versions, authorization metadata and endpoint discovery |
| `GET /v1/decisions` | Recommended catalog endpoint; state-derived actions and revision tokens |
| `POST /v1/decisions/execute` | Recommended execution endpoint; an exact bounded selection and its expected revisions |
| `DCP-Version: 0.1` | Exact version negotiation; provider echoes the selected value |

Discovery may advertise relative endpoints, an execution endpoint and a stream endpoint. Do not hardcode an undiscovered provider URL or infer compatibility between different version strings. An unsupported requested version returns `unsupported_version` with HTTP 406.

Provider-local `/v1/decisions/plan` is an optional extension outside the core 0.1 contract. A plan is a proposal until it becomes a bounded selection and passes provider validation.

## Catalog and action reference

Catalogs carry `catalog_revision` and `state_revision`. They are opaque equality tokens, not sortable version numbers. A known freshness bound may be exposed through `expires_at`.

Each action declares its input schema, availability, permitted lifecycle phases and all four safety properties:

| Property | Meaning |
| --- | --- |
| `idempotent` | Replay with the same request ID has no additional effect |
| `reversible` | A bounded compensating or cancellation action exists |
| `requires_final` | Commit needs final evidence |
| `confirmation_required` | Commit needs explicit confirmation evidence |

Unavailable actions include a machine-readable reason. Availability is advisory: execution validates again. Phase membership determines prepare support; reversibility or idempotence alone does not.

## Lifecycle and execution

1. **Observe:** obtain discovery, a fresh catalog and current evidence.
2. **Speculate:** select an advertised action and arguments matching its schema. Record exact evidence and catalog revisions.
3. **Prepare, if supported:** request reversible intermediate state under the provider's phase and finality policy.
4. **Commit:** supply the exact revisions, authorization, required finality and confirmation. The provider checks these immediately before the effect.
5. **Cancel:** explicitly cancel obsolete preparations, referencing their receipt. Cancellation is idempotent.

Execution requests include `dcp_version`, `request_id`, `decision_id`, `action_id`, `arguments`, `phase`, both expected revision tokens, evidence identity/revision/status and `confirmed`. Prepared operations also refer to their preparation receipt when required. Use the canonical execute schema rather than constructing a permissive replacement.

On `stale_catalog` or `stale_state`, refresh and make a new decision. Do not substitute the new revision into an old decision. A reused request ID with different content is a conflict. Providers return receipts for rejected, no-op and accepted outcomes; preserve the receipt as the operation result.

## Streaming

DCP defines transport-neutral append-only stream messages. Session sequence numbers and evidence revisions advance exactly one step. Evidence carries its complete current value, not a delta. Final and retracted evidence are terminal.

The optional WebSocket binding uses `wss` and subprotocol `dcp.v0.1`, with one JSON message per text frame. Binary frames are undefined. Reconnection starts a new session; replay is not implied. Consumers must fail closed on gaps, duplicates, invalid parents or unknown revisions.

A reconciled decision history records confirmation, replacement, cancellation or compensation. History itself grants no execution authority.

## Provider integration runbook

1. Derive actions from actual provider state and expose their input schemas.
2. Implement exact version negotiation and credential-free discovery.
3. Authenticate execution independently from catalog visibility.
4. Check both opaque revision tokens at the effect boundary.
5. Declare only supported phases and safety properties your implementation actually satisfies.
6. Test stale state, stale catalog, denied authority, duplicate request IDs and incompatible replays.
7. Validate discovery, catalog, execute and receipt fixtures with the owner's conformance suite.
8. Add streaming or parameter rounds only if your provider implements those optional contracts.

In the owner checkout, the existing verification command is:

```sh
npm run test:conformance
```

The required Node version and setup are documented by the owning repository. This guide does not assert that a provider passes conformance because it exposes similarly named endpoints.

## DCP, MCP and Unibus

MCP advertises general tools; DCP advertises current bounded decisions. An application may expose both from a provider-owned registry. [Unibus](https://fungos.dev/projects/unibus.html) can lease and route either contract without translating one into the other. Credentials remain local to the edge; catalogs and capability projections do not contain bearer values.

## Source and status

The specification covers version negotiation, catalog/execute/receipt schemas, streaming, a WebSocket binding and an optional parameter-rounds profile. DCP remains a draft. Check each provider's advertised version and capabilities before integrating it.
