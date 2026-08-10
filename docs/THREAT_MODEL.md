# Threat model

This is the Phase 0 threat model. It must evolve with implementation and be
validated against actual code, packaged artifacts, deployment defaults, and
recovery procedures.

## Security objectives

1. A daemon, proxy, database, or backup compromise does not reveal synchronized
   metadata values or raw mail/account locators.
2. Only authorized non-revoked devices may upload or retrieve workspace data.
3. Invalid or modified operations are detected before client application.
4. Replays and rollback attempts are detected within the accepted v1 model.
5. A sleeping/unavailable daemon cannot cause local data loss or block Brev.
6. Resource abuse is bounded.
7. Operational diagnostics do not become a secondary metadata store.

## Assets

- Workspace root/content keys and recovery material.
- Device signing and pairing private keys.
- Decrypted Brev metadata and its association with a message/account.
- Provider-authoritative state and local Brev state.
- Membership/revocation integrity.
- Operation ordering, cursors, counters, and snapshots.
- Availability and recoverability of the opaque log.
- User privacy against host, proxy, network observer, and other workspaces.

## Adversaries

- Internet client without membership.
- Authorized device that is later lost, stolen, or revoked.
- Malicious or compromised daemon administrator.
- Compromised reverse proxy/tunnel/DNS/TLS endpoint.
- Another workspace on a shared instance.
- Attacker with a copied/restored database or backup.
- Attacker able to crash/restart the daemon or race requests.
- Dependency/build/update supply-chain attacker.
- Curious operator inspecting logs, database, backups, and traffic patterns.

## Trust assumptions

- At least one authorized client and its OS key store remain trustworthy during
  normal recovery.
- Client implementations correctly validate signatures, AEAD, membership,
  versions, locators, and merge clocks.
- Standard cryptographic primitives and random sources behave as specified.
- Users protect their host and recovery material to a reasonable standard.
- Mail providers accurately represent provider-owned state; they are not trusted
  with Brev Vault state beyond normal mail content they already host.

## Threats and planned mitigations

| Threat | Planned mitigation | Residual risk / validation |
| --- | --- | --- |
| Server reads metadata | Client-side AEAD; no server keys; no raw identifiers | Traffic size/timing and pseudonyms remain visible |
| Database/backup theft | Same encrypted envelopes; private keys never stored | Offline traffic analysis and destructive deletion remain possible |
| Unauthorized upload/download | Per-device Ed25519 authentication and membership | Key theft impersonates device until revocation |
| Replay | Monotonic per-device counter atomically stored with accepted batch | Restore/counter rollback design must be proven |
| Envelope tampering | Sign full canonical envelope; AEAD associated data | Canonicalization differences require cross-language vectors |
| Ciphertext substitution | Bind version/workspace/device/op/counter/epoch in signature/AAD | Exact field set must be frozen before code |
| Revoked device continues | Immediate server rejection; rotate future content epoch without re-encrypting retained ciphertext | Previously downloaded plaintext and older epochs already held by the device cannot be revoked |
| Malicious snapshot rollback | Signed snapshots, local checkpoints, retained operation tail | Full equivocation/fork detection may remain out of v1 scope |
| Server omits/reorders operations | Sequence continuity and client checkpoints | Server can deny service; fork detection needs explicit design |
| Wrong message receives state | Multi-locator client match; ambiguity quarantine | Provider quirks require live synthetic-account testing |
| Cross-account correlation | Account-scoped keyed locators kept in ciphertext | Workspace traffic pattern still correlatable at transport layer |
| Oversized/expensive request | Header/body/batch/item limits before allocation; quotas; timeouts | Limits need fuzz/load evidence |
| SQLite corruption/crash | Transactions, WAL, integrity checks, backup/restore tests | Recent uncommitted work may be retried by clients |
| Public endpoint brute force | Strong random identifiers, device auth, rate/concurrency limits | Volumetric denial requires network-layer help |
| Proxy trust confusion | App auth independent of proxy; explicit trusted-proxy config | Misconfigured operator may still expose service |
| Sensitive logs | Logs off/minimal by default; structured allowlist fields | OS/proxy logs remain outside daemon control |
| Malicious dependency/update | Minimal dependencies, pinning, SBOM, signed releases, review | Build system/account compromise remains possible |
| Pairing interception | Authenticated transcript, ephemeral X25519, human fingerprint, expiry | Human approval can still be socially engineered |
| All devices lost | User-held offline recovery secret plus surviving-device recovery; daemon never holds recovery material | Loss of every authorized device and the recovery secret is permanent; recovery-secret compromise can weaken confidentiality |

## Privacy threats

Encryption does not remove all personal-data considerations. An operator may
observe:

- household/device access patterns;
- approximate volume of organized mail;
- when devices are active;
- IP addresses and network geography;
- that multiple devices share one workspace;
- revocation/pairing and snapshot events.

Possible reductions include batching, padding size classes, delayed upload,
minimal retention, and private-network deployment. Each adds battery, latency,
or storage cost; Phase 0 must state the accepted v1 trade-off.

## Availability threats

- Home PC asleep, powered off, disconnected, or behind a changed network.
- Tailscale/VPN/tunnel disabled on a mobile device.
- Database disk full or read-only.
- Failed schema migration.
- Backup restored behind client checkpoints.
- Host loss without backup.
- Malicious host deleting or withholding operations.

Clients mitigate availability by applying locally, persisting pending operations,
retrying idempotently, and supporting reseed from an authorized snapshot.

## Out of scope for v1

- Preventing an authorized device from reading data it was legitimately given.
- Hiding all traffic metadata from a global observer.
- Availability against a malicious host or large DDoS without external
  infrastructure.
- Formal prevention of a server presenting permanent split views unless an
  explicit transparency/gossip design is accepted.
- Protecting a compromised Brev client process after local decryption.
- Synchronizing arbitrary PII-bearing free text.

## Required security verification

- Independent protocol test-vector implementations.
- Parser fuzzing and size-bound tests.
- Race/concurrency and transactional replay tests.
- Crash/restore and counter/nonce regression tests.
- Cross-workspace authorization tests.
- Revocation/key-epoch tests.
- Database/log/package inspection for prohibited plaintext.
- Dependency/license/vulnerability review.
- Public exposure review for each supported deployment profile.
- Backup/restore and client reseed exercise.
- Physical-device mobile test for connectivity/status behaviour.
