# Implementation plan

This plan defines the order and evidence required to build `brev-syncd`. Later
phases must not begin by assumption: each phase has an explicit exit gate.

## Success definition

The first useful release lets one person run a daemon on an ordinary home
computer or server and synchronize pin and flag-colour state among Brev on
macOS, iPhone, and iPad without the daemon receiving mail content, credentials,
raw message locators, or workspace decryption keys.

The system remains useful when the daemon is asleep or unreachable. Each client
applies changes locally, queues encrypted operations durably, and converges when
connectivity returns.

## Phase 0 — Protocol and threat model

### Deliverables

- Accepted trust boundary and data classification.
- Versioned canonical serialization for signed requests and encrypted
  operations.
- Device, workspace, pairing, revocation, recovery, and rekey lifecycle.
- Message-locator construction and ambiguity rules.
- Conflict semantics for pin and flag colour.
- Nonce, key derivation, signature, counter, and rollback rules.
- Server-visible metadata disclosure.
- Synthetic interoperability and abuse-case test vectors.
- Independent security review of the protocol before production code.

### Open decisions

| Decision | Required outcome |
| --- | --- |
| Payload AEAD | One audited cross-platform primitive supported by Swift and Go, with explicit nonce rules |
| Canonical encoding | Deterministic encoding for signatures and test vectors; JSON alone is not canonical without a profile |
| Workspace authority | Define the long-lived workspace signing authority and its recovery/replacement path |
| Pairing | Define online QR pairing, expiry, cancellation, and approval transcript |
| Recovery | Decide whether v1 supports only surviving-device recovery or also an offline recovery secret |
| Revocation | Define membership rejection immediately and content-key rotation for future epochs |
| Counter persistence | Prevent replay-counter or nonce reuse after crash, restore, or device migration |
| Message identity | Define keyed locator inputs, aliasing, moves/copies, missing IDs, and ambiguity quarantine |
| Snapshot trust | Define who may produce a snapshot and how clients reject rollback or incomplete history |
| Padding | Required disclosure: v1 accepts ciphertext-size, timing, operation-volume, and source-address leakage; no padding or traffic-shaping guarantee ([ADR-0002](adr/0002-v1-exposure-and-metadata-leakage.md)) |
| Network exposure | Required: loopback daemon plus a maintained TLS/private-network reverse proxy; no direct public listener in v1 ([ADR-0002](adr/0002-v1-exposure-and-metadata-leakage.md)) |
| Instance tenancy | Required: one owner and one Workspace per v1 instance; schema isolation is not multi-owner support ([ADR-0002](adr/0002-v1-exposure-and-metadata-leakage.md)) |

### Exit gate

- Protocol test vectors are implemented independently in Go and Swift or in two
  isolated reference implementations.
- Negative vectors cover tampering, wrong workspace/device, replay, revocation,
  nonce duplication, rollback, truncation, unknown version, and oversized data.
- Henrik accepts the privacy disclosure and recovery trade-off.
- Security review has no unresolved critical/high design finding.

## Phase 1 — Daemon storage vertical slice

### Scope

Implement the smallest daemon that supports two synthetic clients:

1. initialize a single-owner instance;
2. bootstrap one workspace and first authorized device;
3. admit a second device with an approved membership record;
4. accept a bounded signed opaque operation batch;
5. reject invalid, replayed, unknown, or revoked-device batches;
6. assign monotonically increasing workspace cursors;
7. return deltas after a cursor;
8. store and return an authorized opaque snapshot;
9. back up and restore the database;
10. recover from process restarts without counter or cursor regression.

### Constraints

- Go standard library first; every dependency requires a written reason and
  license/maintenance audit.
- SQLite only.
- Loopback HTTP for tests/development; production access documented through an
  HTTPS terminator.
- No Brev application dependency and no mail-shaped fixtures.
- No web UI, metrics exporter, telemetry, PostgreSQL, Redis, or clustering.

### Verification

- Unit and integration tests for the storage and authorization state machine.
- Race-enabled test suite.
- Fuzzing for envelope parsing and canonical request validation.
- Crash/restart, duplicate request, concurrent writer, migration failure,
  corruption, quota, and backup/restore tests.
- Database/log inspection proves no decrypted domain payload exists.

### Exit gate

Two synthetic clients converge through a restarted daemon while abuse cases
fail closed. Backup/restore and revocation evidence is recorded.

## Phase 2 — Client reference engine

### Scope

- Durable local operation queue.
- Workspace/device key storage abstraction.
- Envelope encryption/decryption and request signing.
- Cursor, retry, bounded backoff, idempotency, and snapshot handling.
- Deterministic merge for pin and flag-colour registers.
- Synthetic account/message locator resolver.
- Reseed a fresh daemon from an authorized client snapshot.

The reference engine may begin as a Swift package intended for Brev, but its
protocol tests must remain reusable without the Brev UI.

### Exit gate

Mac/iOS-compatible Swift tests pass the same protocol vectors as the daemon,
including offline concurrent edits and daemon replacement.

## Phase 3 — Brev integration

### First product slice

- Explicit opt-in Sync Vault setting.
- Add-by-URL and QR pairing.
- Per-device Keychain storage.
- Pin synchronization.
- Flag-colour synchronization only where provider capability policy assigns
  Brev local authority.
- Status: synced, pending, unreachable, revoked, incompatible, recovery needed.
- Account removal clears/reconciles scoped local state without touching provider
  state unexpectedly.

### Identity acceptance matrix

- RFC 8474 EMAILID available.
- RFC Message-ID available and unique.
- Duplicate RFC Message-ID.
- Missing RFC Message-ID.
- UIDVALIDITY change.
- Folder rename.
- Message move and copy.
- Same message present in multiple folders/accounts.
- Provider state contradicts local flag colour.

Ambiguous identity must remain pending/unresolved; it must never apply metadata
to a probable match.

### Platform acceptance

- macOS to iPhone and iPad in both directions.
- Offline edit on every platform and deterministic convergence.
- App termination/relaunch with pending work.
- Daemon unreachable/sleeping and later recovery.
- Physical-device verification for iPhone and iPad.
- No promise of exact background delivery time.

### Exit gate

Pin and flag colour pass the identity and platform matrices with rendered status
UI, privacy documentation, and Henrik's physical-device acceptance.

## Phase 4 — Deployment and operational hardening

### Deliverables

- Reproducible native binaries for Linux, macOS, and Windows.
- Multi-architecture non-root OCI image with pinned base/digest policy.
- Docker Compose example.
- systemd, launchd, and Windows service examples.
- Tailscale Serve, Cloudflare Tunnel, and Caddy deployment profiles.
- Health endpoint, local status CLI, backup, restore, migration, revoke, and
  rotate commands.
- Signed releases, checksums, SBOM, vulnerability scan, and upgrade runbook.
- Data retention/compaction policy and capacity evidence.

### Exit gate

Each supported topology passes an install, restart, upgrade, backup/restore,
revocation, and cross-device smoke test. Public exposure passes a focused
security review.

## Phase 5 — Additional typed metadata

Candidate fields are added one at a time only after privacy, identity, provider
authority, merge, removal, and UI semantics are defined:

- Done;
- snooze timestamp;
- follow-up timestamp;
- thread watch/mute;
- local classification override;
- favourite-folder mapping through opaque locators;
- keep-offline intent with a device-local storage override;
- carefully enumerated non-sensitive presentation settings.

Free-form notes, signatures, aliases, rules, saved searches, VIP lists,
allowlists, subjects, and correspondents are excluded from the strict metadata
profile. Supporting any of them would require a separate privacy mode and
decision.

## Deferred options

- Optional opaque APNs wake relay.
- Multiple owners/families per workspace.
- Multiple independent workspaces per shared instance beyond schema isolation.
- PostgreSQL or high availability.
- Hosted managed service.
- Browser administration.
- Strict traffic-analysis padding beyond the accepted v1 profile.
