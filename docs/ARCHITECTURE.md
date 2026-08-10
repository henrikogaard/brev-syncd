# Architecture

## System role

`brev-syncd` is a synchronization mailbox for encrypted Brev metadata. It is
not a mail server, mail proxy, provider adapter, identity provider, or source of
message truth.

```text
                         server-visible boundary
                                  |
  Brev client                       |        brev-syncd
  -------------------------------   |   -------------------------------
  mail provider adapter             |   workspace/device public records
  local message cache               |   replay counters and quotas
  message identity resolver         |   opaque operation sequence
  typed metadata store              |   opaque snapshots
  conflict merge engine             |   SQLite and backup/restore
  encryption and signing         --HTTPS-- request validation
  pending-operation queue           |   no domain payload interpretation
                                  |
```

## Trust boundaries

### Trusted by a Brev client

- Its local operating-system key store.
- Its own process and local metadata database.
- Authorized workspace/device keys after pairing.
- Mail-provider results only for the provider-owned state they represent.

### Not trusted with plaintext domain data

- `brev-syncd` host and database.
- Tailscale/Cloudflare/reverse proxy/DNS/TLS infrastructure.
- Backups of the daemon database.
- Other devices until explicitly admitted.

TLS protects transport and application authentication. Payload encryption is a
separate layer and must remain secure if TLS terminates at a proxy.

## Components

### Brev-side sync engine

Responsibilities:

- Map a local account to a random workspace-local account identity.
- Construct keyed message locators from locally available provider identity.
- Apply provider-authority policy before producing a Brev metadata operation.
- Encrypt/sign operations and persist them locally before upload.
- Pull/decrypt/validate remote operations.
- Resolve locator aliases against the local mail cache.
- Merge state deterministically and quarantine ambiguity.
- Produce encrypted snapshots and restore/reseed another daemon.
- Surface pending, unreachable, revoked, incompatible, and recovery states.

### Transport adapter

The client uses one versioned HTTPS API regardless of whether connectivity is
provided by LAN, Tailscale, Cloudflare Tunnel, Caddy, or a VPS. Network-specific
identity may add policy but cannot replace device signatures.

### Daemon service

Responsibilities:

- Validate protocol version, request bounds, device membership, signatures, and
  replay counters.
- Atomically persist opaque batches.
- Assign monotonically increasing sequence numbers within a workspace.
- Return bounded deltas after an authorized cursor.
- Store/serve authorized opaque snapshots.
- Revoke devices and enforce the current membership view.
- Enforce instance/workspace/device quotas.
- Perform migrations, status, backup, restore, and health checks.

The daemon must not branch on encrypted metadata kind or message identity.

## Proposed storage model

Names are illustrative until Phase 1 schema review.

```text
instances
  schema_version
  instance_id
  initialized_at

workspaces
  workspace_id
  authority_public_key
  current_membership_generation
  created_at

devices
  workspace_id
  device_id
  signing_public_key
  membership_generation
  last_accepted_counter
  status

operations
  workspace_id
  sequence
  operation_id
  device_id
  device_counter
  protocol_version
  ciphertext
  signature
  accepted_at

snapshots
  workspace_id
  snapshot_id
  through_sequence
  creator_device_id
  membership_generation
  ciphertext
  signature
  accepted_at
```

Database timestamps are operational metadata, not merge authority. Clients use
authenticated clocks/counters inside encrypted payloads for domain merges.

## Server-visible information

The first privacy disclosure must enumerate at least:

- instance/workspace/device pseudonyms and public signing keys;
- membership and revocation timing;
- request source IP at the socket/proxy layer;
- request timing, frequency, sizes, result codes, and operation counts;
- per-workspace cursors and device counters;
- ciphertext, nonces if carried outside it, signatures, and protocol versions;
- snapshot frequency and approximate encrypted state size;
- backup and retention timing.

Access logging is disabled by default, but absence of logs does not mean the
network stack cannot observe these values while serving a request.

V1 explicitly accepts this metadata leakage. It does not promise ciphertext
padding, delayed uploads, traffic shaping, anonymity, or resistance to a
traffic-analysis observer. See
[ADR-0002](adr/0002-v1-exposure-and-metadata-leakage.md).

## Client-encrypted domain model

An encrypted operation is expected to contain:

```text
schema version
workspace/account pseudonym
logical record identifier
keyed locator set and aliases
typed property name
typed value or tombstone
logical clock and writer device
key epoch
```

MVP typed properties:

- `pin`: Boolean last-writer register;
- `flagColor`: bounded enum-or-null last-writer register.

Provider `isFlagged == false` remains authoritative for flagged-ness and may
cause the Brev client to clear local colour according to Brev's provider policy.

## Message identity

Clients build a set of secret-keyed locators, preferring all available stable
signals rather than trusting one universal identifier:

1. RFC 8474 EMAILID/provider stable immutable ID when supported;
2. normalized RFC Message-ID when present;
3. mailbox identity + UIDVALIDITY + UID for exact IMAP location;
4. a reviewed keyed fallback fingerprint when necessary.

Raw inputs stay local. The encrypted operation carries only keyed outputs. A
logical record may gain locator aliases during move/copy reconciliation.

Matching rules must account for:

- absent or duplicate Message-ID;
- messages copied to multiple folders;
- message moves with a new UID;
- folder rename and UIDVALIDITY reset;
- identical messages in different accounts;
- provider-specific stable IDs;
- local cache eviction and later re-resolution.

No-match remains pending. Multiple plausible matches are quarantined. “Best
guess” mutation is forbidden.

## Availability and recovery

The daemon is not on Brev's critical local path:

- local changes apply immediately;
- encrypted operations are durably queued before upload;
- retry is bounded and battery-aware;
- a sleeping home host is normal;
- a restored daemon resumes from its durable sequence/counter state;
- an authorized client can reseed a new daemon from a client-created snapshot.

Server backup without a surviving workspace key protects availability, not
plaintext recovery. Losing all authorized devices and recovery material can be
permanent.

## Multi-workspace posture

Schema and authorization isolate workspaces from the beginning. V1 supports one
owner and one Workspace per daemon instance; it does not offer shared or
multi-owner operation. Public unauthenticated workspace registration is out of
scope. Supporting multiple unrelated owners later requires a separate privacy,
administration, and abuse review.
