# Worklog

## 2026-08-02 — Codex — Repository planning bootstrap

**Goal:** Create the public `brev-syncd` repository with enough durable context
for another agent to begin the protocol-design phase without relying on chat
history.

**Summary:**

- Initialized the public AGPL-3.0 repository.
- Added the canonical agent contract, architecture, protocol, threat-model,
  deployment, phased-plan, resource, contribution, security, and handover
  documents.
- Kept the repository documentation-only; no daemon or Brev client code exists.

**Verification:**

- `git diff --cached --check` passed.
- Every relative Markdown link resolved locally.
- Every linked Brev source/ADR path existed on current `main`.
- The documentation scan found no email-address-shaped fixture or credential.
- GitHub read-back confirmed the public repository, focused topics, Issues,
  automatic merged-branch deletion, and private vulnerability reporting.

**Skipped:** No Go tests, build, container, protocol interoperability test, or
security validation exists because this bootstrap is documentation-only.

**Next:** Complete the Phase 0 decisions in `docs/HANDOVER.md`, create synthetic
protocol test vectors, and obtain focused security review before scaffolding the
daemon.
# 2026-08-10 — Agent — Phase 0 workflow conventions

## Goal

Make the GitHub issue workflow and ubiquitous language explicit before creating
the Phase 0 protocol decision map.

## Changes

- Recorded GitHub Issues as the planning tracker, including Wayfinder map,
  dependency, claim, and resolution conventions.
- Added the repository triage-label vocabulary and the Phase 0 Wayfinder label
  vocabulary.
- Added the root domain glossary and an ADR location rule.
- Linked the conventions from `AGENTS.md` and the repository map.

## Verification

- Read back every new convention file.
- Verified all documented local Markdown links resolve.
- Confirmed the required GitHub labels exist after creation.

## Skipped

- No product, protocol, daemon, or client code exists; no build or test suite
  applies to this documentation-only change.

## Next

Create the Phase 0 Wayfinder map and begin with the recovery/revocation
decision, not service scaffolding.

# 2026-08-10 — Agent — Phase 0 decision map

## Goal

Create the GitHub planning map that holds the remaining Phase 0 decisions
without beginning daemon implementation.

## Changes

- Created [Phase 0 protocol decision map](https://github.com/henrikogaard/brev-syncd/issues/3).
- Added its four native child decision issues: recovery/revocation, metadata
  leakage/network exposure, mail-locator/merge safety, and Phase 0 review
  evidence.
- Corrected three duplicate setup issues by detaching, marking, and closing
  them; the map contains only the canonical child issues.

## Verification

- Read back the map and its native child-issue list through the GitHub API.
- Confirmed the documentation branch remains clean after its workflow commit.

## Next

Henrik answers the four frontier decisions. Do not claim or resolve a decision
issue until its explicit claim has been posted and verified.

# 2026-08-10 — Agent — Recovery and revocation decision

## Goal

Record Henrik's accepted v1 recovery and revoked-device boundary from the Phase
0 decision map.

## Changes

- Added ADR-0001: offline recovery secret, no daemon-held recovery material,
  immediate request rejection after revocation, and future-epoch rotation only.
- Updated the protocol, plan, threat model, architecture, and handover to match
  the decision.

## Verification

- `git diff --check`.
- Local Markdown-link inspection.

## Skipped

- Documentation-only protocol decision; no daemon, client, cryptographic
  primitive, or test-vector implementation exists yet.

## Next

Define recovery-package bytes, authority replacement, epoch distribution, and
negative vectors only after the remaining Phase 0 decisions are resolved.

# 2026-08-10 — Agent — V1 exposure and metadata-leakage decision

## Goal

Record Henrik's accepted v1 transport exposure, privacy disclosure, and
instance-ownership boundary from the Phase 0 decision map.

## Changes

- Added ADR-0002: loopback daemon, maintained TLS/private-network proxy for
  reachability, no direct public listener, no padding guarantee, and one owner
  and Workspace per instance.
- Updated deployment, architecture, threat model, plan, and handover guidance.

## Verification

- `git diff --check`.
- Local Markdown-link inspection.

## Skipped

- Documentation-only protocol decision; no listener, proxy, daemon, or client
  implementation exists yet.

## Next

The map can next resolve client-only mail-locator and merge-safety semantics.

# 2026-10-11 — Agent — Rebase recovery decision onto main

## Goal

Make the recovery and revocation documentation branch mergeable after
`docs/phase0-workflow` landed on `main`.

## Changes

- Merged `origin/main` into `docs/recovery-revocation`.
- Kept the Phase 0 workflow, decision-map, and recovery worklog entries in
  chronological order.

## Verification

- `git diff --check`.
- Confirmed no remaining conflict markers.

## Skipped

- Documentation-only merge; no daemon, client, or test-vector implementation.

## Next

Keep the exposure-profile branch coherent with this recovery decision.
