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
