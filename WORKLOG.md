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
