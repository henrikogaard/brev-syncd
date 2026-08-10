# Draft protocol v0

This document is a design candidate for Phase 0. It is not an implemented or
stable public protocol. Byte-level choices must be locked by test vectors before
the daemon is scaffolded.

## Goals

- Authenticate every device request independently of the network perimeter.
- Store and retrieve opaque metadata without server-side domain interpretation.
- Support offline writers and deterministic client-side convergence.
- Reject replay, tampering, revoked devices, unknown versions, and oversized
  input.
- Allow backup/restore and migration to another daemon.
- Keep the protocol small enough for independent Swift and Go implementations.

## Non-goals

- Mail-provider access or message synchronization.
- Server-side field queries, search, merge, push content, or message lookup.
- Anonymous transport.
- Byzantine consensus or a formally verified transparency log in v1.
- Arbitrary client-defined records.

## Terminology

| Term | Meaning |
| --- | --- |
| Instance | One running daemon and its database |
| Workspace | One isolated encrypted synchronization domain |
| Authority | Long-lived signing authority that admits/revokes devices |
| Device | One authorized Brev installation with its own signing key |
| Membership generation | Monotonic authorization revision for a workspace |
| Device counter | Monotonic request/operation counter used for replay rejection |
| Server sequence | Monotonic daemon-assigned cursor within a workspace |
| Operation | Signed opaque encrypted domain mutation |
| Snapshot | Signed opaque client-created state through a server sequence |
| Key epoch | Client-side encryption generation after rotation |

## Candidate cryptographic suite

This suite is proposed because standard implementations exist in Apple
CryptoKit and Go. It remains subject to Phase 0 review.

- Device and workspace-authority signatures: Ed25519.
- Pairing key agreement: X25519.
- Key derivation: HKDF-SHA-256 with versioned ASCII context labels.
- Locator derivation: HMAC-SHA-256 under an account-scoped locator key.
- Payload encryption: one reviewed AEAD common to Swift and Go. AES-256-GCM and
  ChaCha20-Poly1305 are candidates; choose exactly one for v0.
- Randomness: operating-system cryptographic random source only.

Do not implement both AEADs as configurable v0 options. One suite reduces
downgrade, negotiation, and test surface.

### Candidate key tree

Names are conceptual, not finalized byte encodings:

```text
workspace_root_key
  -> payload_epoch_key(epoch)
  -> account_namespace_key
       -> account_locator_key(account-local canonical identity)
  -> snapshot_key(epoch)
  -> pairing_wrap_key(pairing transcript)
```

Signature keys are asymmetric device/authority identities and are not derived
from a shared symmetric root unless a reviewed design explicitly requires it.

Each HKDF use must define salt, input keying material, `info` bytes, output
length, versioning, and test vectors. Labels must be domain-separated.

## Canonical signed request

Every protected request should sign a deterministic byte string covering:

```text
protocol version
HTTP method
canonical path
canonical query
workspace ID
device ID
membership generation
device counter
body digest
```

Open questions:

- exact binary/canonical JSON/CBOR representation;
- handling of percent encoding and empty query parameters;
- whether timestamp is included in addition to a counter;
- batching relationship between request counter and operation counters.

Reverse proxies must not be able to alter a signed semantic field. Hop-by-hop
headers and client IP are not part of the application signature.

## Opaque operation envelope

Candidate server-visible fields:

```json
{
  "version": 0,
  "workspace_id": "random-base64url",
  "operation_id": "random-base64url",
  "device_id": "random-base64url",
  "membership_generation": 1,
  "device_counter": 42,
  "nonce": "base64url-if-suite-requires-external-nonce",
  "ciphertext": "base64url",
  "signature": "base64url"
}
```

The signature must cover the full envelope and protocol-specific associated
data. The encrypted payload—not the envelope—contains the message locator,
metadata kind/value, logical clock, tombstone, and key epoch.

Unknown envelope fields and versions should fail closed in v0 unless canonical
encoding explicitly defines safe extension handling.

## Encrypted operation payload

Conceptual plaintext before client-side AEAD:

```json
{
  "schema": 0,
  "account": "keyed-account-pseudonym",
  "record": "random-logical-record-id",
  "locators": ["keyed-locator"],
  "property": "pin",
  "value": true,
  "clock": {"logical": 17, "device": "device-pseudonym"},
  "epoch": 1
}
```

This JSON is illustrative only; it is not the canonical encoding.

Allowed v0 properties:

| Property | Value | Merge |
| --- | --- | --- |
| `pin` | Boolean | Last-writer register by authenticated logical clock, deterministic device tie-break |
| `flagColor` | Bounded stable enum or null | Same register rule; provider reconciliation remains client-side |

The merge rule must not trust wall-clock time alone. The exact logical-clock
advance and tie-break behaviour requires vectors for concurrent offline writes.

## Message locator construction

Locators are computed and consumed only by clients. Candidate domain-separated
inputs include:

```text
HMAC(account_locator_key, "emailid\0" || provider_email_id)
HMAC(account_locator_key, "message-id\0" || normalized_rfc_message_id)
HMAC(account_locator_key, "imap-location\0" || mailbox_id || uidvalidity || uid)
HMAC(account_locator_key, "fallback-v1\0" || reviewed_canonical_fingerprint)
```

Requirements:

- Exact normalization must be specified and tested; do not casually lowercase
  or rewrite Message-ID components.
- Account keys prevent cross-account/cross-workspace equality.
- Folder names and raw values never enter the server-visible envelope.
- Move/copy reconciliation may append aliases to an existing logical record.
- A locator collision or ambiguous local match is quarantined.

## Workspace bootstrap

Proposed flow:

1. `brev-syncd init` creates an instance identity and one-time bootstrap
   capability, shown locally once.
2. First Brev client generates workspace encryption material, workspace
   authority key, and its device signing key.
3. Client submits workspace public metadata plus a bootstrap proof.
4. Daemon stores only the workspace authority public key, first device public
   key/membership certificate, and random workspace/device IDs.
5. Bootstrap capability is consumed and public workspace registration closes.

The bootstrap capability is not a content-encryption or recovery key.

## Additional-device pairing

Candidate online QR flow:

1. New device creates an ephemeral X25519 key and a device signing key.
2. It requests a short-lived pairing record from the daemon.
3. Existing authorized device scans/receives a QR containing the instance URL,
   workspace pseudonym, pairing ID, ephemeral public key, and transcript hash.
4. Existing device verifies human-visible instance/workspace fingerprints.
5. Existing device signs a membership certificate and encrypts the workspace
   key package to the new device's ephemeral key.
6. Daemon records the authorized device membership but cannot decrypt the key
   package.
7. New device confirms possession and pairing record expires.

Required failure behaviour: expiry, cancellation, replay, approval by revoked
device, transcript substitution, multiple approvals, abandoned request, and
wrong-instance QR.

## Revocation and rekey

Server-side revocation immediately rejects subsequent requests from the device.
It does not erase information already downloaded by that device.

Client-side future secrecy requires a new payload key epoch distributed only to
remaining authorized devices. Phase 0 must decide:

- whether retained snapshots/operations are re-encrypted;
- how offline authorized devices receive the new epoch;
- how clients distinguish revoked-device operations accepted before revocation;
- whether one authority device is special or authority can rotate.

## Push/pull operations

Candidate endpoints:

```text
POST /v1/workspaces
POST /v1/pairings
POST /v1/pairings/{id}/approve
POST /v1/pairings/{id}/complete
POST /v1/operations/batch
GET  /v1/operations?after=<sequence>&limit=<bounded>
POST /v1/snapshots
GET  /v1/snapshots/latest
POST /v1/devices/{id}/revoke
GET  /healthz
```

Endpoint names are provisional. Each endpoint needs:

- authentication state;
- signed fields;
- idempotency rule;
- maximum encoded/decoded size;
- maximum item count;
- transaction boundary;
- stable error code;
- rate-limit behaviour;
- privacy-safe diagnostic behaviour.

The server must commit an accepted operation batch and associated device counter
atomically. Retrying the same operation ID may return its prior sequence; it
must not create a duplicate mutation.

## Snapshots and compaction

Snapshots are client-produced encrypted state through a declared server
sequence. The daemon cannot verify domain completeness, so authorization,
signatures, client checkpoint history, and retention rules matter.

Phase 0 must specify:

- authorized snapshot producers;
- whether multiple device snapshots coexist;
- minimum retained operation tail;
- client acknowledgements and safe deletion;
- restored-server rollback detection;
- response when a snapshot is corrupt or undecryptable;
- reseeding a new daemon without trusting the old daemon.

Do not compact solely because one client uploaded a snapshot.

## Errors

Errors should use a small stable code set and omit reflected input:

```text
unsupported_version
malformed_request
request_too_large
unauthorized_device
revoked_device
invalid_signature
replayed_counter
membership_generation_mismatch
cursor_too_old
quota_exceeded
temporarily_unavailable
internal_error
```

Public responses must not distinguish whether an unknown workspace or device
exists more than required for safe client recovery.

## Initial limits to decide before code

- Maximum HTTP body and header bytes.
- Maximum operations per batch.
- Maximum ciphertext per operation.
- Maximum delta page.
- Maximum pending pairings and pairing lifetime.
- Maximum devices per workspace.
- Workspace storage quota and snapshot count.
- Request concurrency per instance/workspace/device.
- Database busy timeout and transaction duration.

Limits require abuse tests and must be configurable only within safe bounds.

## Versioning

- Protocol version is explicit and authenticated.
- Unknown major versions fail closed.
- Migrations never reinterpret an old ciphertext payload server-side.
- Clients must report incompatible daemon/client versions clearly.
- Compatibility policy and supported-version window must be defined before the
  first release.
