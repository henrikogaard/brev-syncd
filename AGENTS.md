# AGENTS.md

This is the canonical working contract for agents in `brev-syncd`. Direct user
instructions override this file. Update this file first if repository workflow
rules change.

## Current stage

The repository is in **Phase 0: protocol and threat-model design**. There is no
daemon implementation yet. Read these files before proposing or writing code:

1. `README.md`
2. `docs/HANDOVER.md`
3. `docs/PLAN.md`
4. `docs/ARCHITECTURE.md`
5. `docs/PROTOCOL.md`
6. `docs/THREAT_MODEL.md`
7. `docs/DEPLOYMENT.md`
8. `docs/RESOURCES.md`

Do not treat draft cryptographic choices as accepted merely because they are
written down. Resolve the Phase 0 gates in `docs/PLAN.md` first.

## Product invariant

`brev-syncd` is an authenticated opaque metadata relay. It must never require or
accept:

- IMAP, SMTP, JMAP, CalDAV, or CardDAV credentials;
- email addresses, correspondent fields, subjects, snippets, bodies, drafts,
  attachments, notes, signatures, rules, or saved searches;
- cleartext RFC Message-ID, account identity, folder name, mailbox identifier,
  UID, UIDVALIDITY tuple, provider object ID, or Brev message ID;
- a workspace content-encryption key or recovery secret.

If a proposed feature needs any of those values on the server, stop and request
an architectural decision. Do not add a hidden exception.

The daemon may necessarily observe transport metadata such as source IP,
request timing, workspace/device pseudonyms, ciphertext sizes, and operation
counts. Documentation must describe that honestly; never claim the service is
anonymous or that encrypted/pseudonymous data is categorically non-personal.

## Authority boundaries

- Mail providers remain authoritative for state they support and synchronize.
- Brev clients own message matching, metadata semantics, encryption, conflict
  resolution, local persistence, and provider reconciliation.
- The daemon owns device admission enforcement, replay rejection, bounded
  durable storage, monotonically ordered cursors, snapshot retention, and
  opaque retrieval.
- A reverse proxy or private network is defence in depth, never the sole
  application authentication layer.

## Engineering rules

- Prefer the smallest implementation that proves the current phase.
- Use standard, reviewed cryptographic primitives through maintained libraries.
  Do not invent cryptography, wire formats, signatures, key derivation, or
  recovery schemes.
- Never reuse an AEAD nonce under the same key. Nonce generation and persistence
  must have explicit tests, including restore and crash scenarios.
- Verify signatures before accepting or persisting an operation.
- Bind to loopback by default. Refuse plaintext non-loopback production
  listeners.
- Disable access logs by default. Structured diagnostics must not contain
  authorization material, ciphertext, operation IDs, workspace IDs, device
  public keys, source addresses, or request bodies unless an explicitly
  documented local diagnostic mode safely redacts them.
- Bound request bodies, batch sizes, ciphertext size, headers, concurrency,
  database growth, and time spent on untrusted input.
- Use constant-time comparisons for secrets and authenticators.
- Fail closed on unknown protocol versions, invalid membership, replayed
  counters, invalid signatures, malformed envelopes, and migration failures.
- Keep schema migrations forward-only, transactional, versioned, and tested
  against the previous supported release.
- Do not add telemetry, analytics, remote crash reporting, or mandatory cloud
  dependencies.
- Do not add a web administration surface during the initial phases.

## Go implementation direction

The intended daemon is a small Go module with this eventual shape:

```text
cmd/brev-syncd/        process entry point
internal/api/          HTTP routing, limits, errors, protocol negotiation
internal/auth/         device signatures, membership, replay protection
internal/store/        storage interface and SQLite implementation
internal/service/      bounded orchestration; no payload interpretation
internal/config/       flags/environment validation
internal/testkit/      deterministic protocol fixtures and fake clients
migrations/            embedded forward-only SQL migrations
```

This is a direction, not permission to scaffold every package at once. The
first implementation PR should include only the vertical slice required by the
accepted Phase 0 protocol.

## Test discipline

Use red-green-refactor TDD for behaviour changes:

1. Add the smallest failing test that proves the intended boundary.
2. Run it and record the expected failure.
3. Implement the smallest passing change.
4. Keep the focused test green while refactoring.
5. Run the broader checks appropriate to the changed surface.

Expected checks once Go code exists:

```sh
go test ./...
go test -race ./...
go vet ./...
govulncheck ./...
```

Security-sensitive code also needs negative and abuse cases: invalid signatures,
replays, rollback/restored counters, oversized input, truncated bodies,
concurrent writers, migration failure, database corruption, unauthorized
devices, and revoked devices.

No live mail credentials are required or permitted for daemon tests. Protocol
fixtures must be synthetic and contain no real user data.

## Git workflow

- Before editing, run `git status --short --branch`, confirm the repository root,
  current branch, remote, and whether the branch already has a PR.
- Branches: `feature/<name>`, `fix/<name>`, `docs/<name>`, or `chore/<name>`.
- Preserve unrelated work. Never reset, rebase, rewrite history, force-push,
  discard changes, or use a shared stash without explicit authorization.
- Stage explicit paths only; never use `git add .` or `git add -A`.
- Use Conventional Commits.
- Push completed changes and open a PR to `main`. Never merge without Henrik's
  explicit authorization after review.
- A clean local test is implementation evidence, not merge, release, deployment,
  or user acceptance.

## Pull requests

PRs must state:

- phase and scope;
- security/privacy boundary affected;
- tests and exact commands run;
- documentation updated;
- skipped checks and why;
- remaining risks or decisions;
- whether protocol compatibility or a migration is involved.

Protocol, cryptography, membership, recovery, storage schema, network exposure,
logging, dependency, and license changes require focused review. Do not combine
them with unrelated cleanup.

## Documentation and worklog

Update canonical documentation with behaviour or contract changes:

- architecture/trust changes: `docs/ARCHITECTURE.md`;
- wire/device changes: `docs/PROTOCOL.md`;
- threats or mitigations: `docs/THREAT_MODEL.md`;
- install/network changes: `docs/DEPLOYMENT.md`;
- phase status and gates: `docs/PLAN.md`;
- session state and handoff: `docs/HANDOVER.md` and `WORKLOG.md`.

Append a concise `YYYY-MM-DD — Agent — scope` entry to `WORKLOG.md` for any
repository-state change. Record the goal, files changed, verification, skipped
checks, blockers, and next action. Never include secrets or raw sensitive logs.

## Delivery terms

- **Implemented locally:** intended behaviour exists and focused verification
  passes.
- **Ready for review:** committed, pushed, PR open to `main`, and evidence is in
  the PR.
- **Merge-ready:** required CI/reviews and applicable security validation pass.
- **Done:** merged and every acceptance, release, deployment, or human-signoff
  gate required by the task is satisfied.

## Licensing

Daemon code is AGPL-3.0. Preserve the license and notices. Third-party code must
be compatible with AGPL distribution, recorded in third-party notices, pinned,
and reviewed before adoption. The Brev application is a separate MIT-licensed
project; network interoperability does not change either repository's license.
