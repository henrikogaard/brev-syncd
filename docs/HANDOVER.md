# Handover

## Bottom line

The public `brev-syncd` repository has been initialized as an AGPL-3.0 planning
project. No daemon, Go module, API, database, container, release, or Brev client
integration exists yet.

The next agent should work only on **Phase 0 protocol decisions and synthetic
test vectors**. Do not scaffold the complete service or integrate Brev until the
Phase 0 exit gate in [`PLAN.md`](PLAN.md) is satisfied.

## Confirmed direction

- The Brev application remains MIT.
- The daemon is AGPL-3.0 from its first commit.
- The daemon can run on a user's PC; cloud hosting is optional.
- A sleeping/offline host delays convergence but does not block local changes.
- Tailscale is the recommended home deployment; Cloudflare Tunnel and direct
  HTTPS remain supported alternatives.
- Mail providers remain authoritative for supported provider state.
- First synchronized Brev state is pin and provider-fallback flag colour.
- The server never connects to mail providers and never receives mail
  credentials, cleartext content, or raw message locators.
- Meaningful merge and identity decisions happen on clients.
- The daemon persists authenticated opaque operations and snapshots in SQLite.
- V1 requires a user-held offline recovery secret as well as surviving-device
  recovery; the daemon never receives recovery material.
- Revocation immediately blocks new requests and protects future payload-key
  epochs only. Retained ciphertext is not re-encrypted in v1.

## Important precision

“Zero-knowledge payload” means the daemon cannot decrypt domain payloads. It is
not a formal zero-knowledge proof protocol and not a claim of anonymity. A host
or proxy can still observe IP addresses, timing, request sizes, workspace/device
pseudonyms, and operation volume.

Raw RFC Message-ID, folder, UID, UIDVALIDITY, provider object ID, Brev message
ID, and account identity must not be sent even inside a merely pseudonymous
server index. Clients should compute keyed locators locally and keep them inside
the encrypted payload. The server retrieves by workspace cursor, not message.

## Exact next task

Create a Phase 0 PR that turns the candidate protocol in [`PROTOCOL.md`](PROTOCOL.md)
into executable synthetic test vectors without starting the HTTP daemon.

The PR should:

1. resolve or clearly narrow the AEAD and canonical-encoding decision;
2. define byte-level request-signature input;
3. define workspace/device key derivation labels;
4. define operation and snapshot associated data;
5. define pairing and membership certificate bytes;
6. define counter persistence and replay rejection across restore;
7. define pin and flag-colour merge examples;
8. define locator examples using entirely fictional values;
9. add positive and negative fixture files;
10. include a small Go verification tool and an independent Swift verifier, or
    explain why independence is not yet achieved.

The test vectors must not contain real emails, message IDs, domains, device keys,
or production data.

## Decisions still requiring Henrik

- Accepted ciphertext padding/traffic-analysis leakage for v1.
- Whether one daemon instance may host multiple unrelated owners at v1.
- Whether a public direct listener is supported by the daemon itself or only
  through a documented TLS reverse proxy.
- Final security reviewer and acceptance evidence for Phase 0.

## Known risks to keep visible

- A compromised daemon can delete, withhold, reorder, or fork views even when it
  cannot decrypt payloads. Local checkpoints and signatures reduce silent
  rollback but do not automatically solve server equivocation.
- Device revocation cannot make a previously authorized device forget data it
  already decrypted.
- Message-ID can be missing or duplicated; IMAP UID identity is mailbox- and
  UIDVALIDITY-scoped and changes across moves/copies.
- iOS/iPadOS background scheduling is opportunistic. Foreground convergence is
  the reliable baseline.
- Tailscale and Cloudflare reduce exposure but introduce their own control-plane
  or transport metadata. Application authentication remains mandatory.
- Losing every authorized device and recovery secret makes encrypted server
  data permanently unrecoverable.

## Suggested first implementation constraints

- Go, one process, one SQLite database.
- One owner/workspace in user experience; workspace isolation in the schema.
- Loopback bind by default.
- JSON API with a canonical signed sub-format or another explicitly canonical
  encoding.
- No admin website.
- No mail-shaped server model.
- No metrics/telemetry exporter.
- No dependency until its need, maintenance, and AGPL compatibility are written
  down.

## Verification expected before handoff to Phase 1

- Protocol vector tests run in both implementations.
- Tampered, replayed, revoked, wrong-workspace, unknown-version, truncated, and
  oversized vectors fail.
- Nonce/counter restore cases are explicit.
- A privacy inspection enumerates every server-visible field.
- `git status --short` is clean.
- The branch is pushed with a PR to `main` containing commands and residual
  risks.
