# Issue tracker

`brev-syncd` uses GitHub Issues as its planning tracker. The canonical remote is
[`henrikogaard/brev-syncd`](https://github.com/henrikogaard/brev-syncd).

## Wayfinder maps

A long-running planning effort has one GitHub Issue labelled `wayfinder:map`.
Its body follows the Wayfinder map structure: Destination, Notes, Decisions so
far, Not yet specified, and Out of scope.

Each decision is a child issue with one `wayfinder:<type>` label. Use GitHub's
native sub-issue relationship when it is available to the acting account. Use
GitHub's native issue dependency relationship for blocking; do not imitate a
dependency with only prose in an issue body. If either native relationship is
unavailable through the available client, stop and record that operational
blocker on the map rather than silently inventing a second tracker.

## Claiming a decision

Before working an unblocked decision, post a comment in this exact form:

```text
Claimed by: <agent task name>
Purpose: <one-line decision scope>
```

Reload the issue and verify that this is the latest active claim. If another
active claim exists, do not work the issue. Close a claim with either a
resolution comment or a brief release comment.

## Resolving a decision

Post the decision, evidence, residual risk, and links to resulting artifacts as
a resolution comment. Close the child issue only after the decision holder has
accepted it. Then add a one-line linked gist to the map's **Decisions so far**
section. A closed issue that is beyond the map destination belongs in **Out of
scope**, not **Decisions so far**.

## Phase 0 guardrail

Phase 0 issues resolve protocol and threat-model decisions or produce synthetic
test vectors. They do not authorize a daemon, HTTP API, database, container, or
Brev integration implementation.
