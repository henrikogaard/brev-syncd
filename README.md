# brev-syncd

`brev-syncd` is the planned zero-knowledge-payload metadata relay for
[Brev](https://github.com/henrikogaard/brev). It lets a user's Brev clients
converge on Brev-only state without giving the daemon mail credentials,
message content, headers, addresses, folder names, or raw message identifiers.

> **Status: planning only.** No usable daemon, protocol implementation, or
> Brev client integration exists yet. The documents in this repository are a
> reviewed starting point, not a security claim or released specification.

## Product boundary

The mail provider remains authoritative for provider-supported state such as
IMAP `\Seen`, `\Flagged`, `\Answered`, persistent keywords, and provider
labels. `brev-syncd` is intended for Brev-owned metadata that the provider
cannot safely or consistently synchronize, beginning with:

- pin / unpin;
- flag colour when the provider cannot persist it;
- later, explicitly reviewed state such as Done, snooze, follow-up, thread
  watch/mute, classification overrides, and keep-offline intent.

The daemon does not connect to IMAP or SMTP. Brev clients resolve message
identity locally, encrypt metadata locally, merge conflicts locally, and use
the daemon only as an authenticated append-only store for opaque envelopes.

## Design principles

1. **Local-first:** Brev works when the daemon is absent. Changes queue locally
   and converge later.
2. **Zero-knowledge payload:** the daemon cannot decrypt the synchronized
   metadata. This is a product description, not a claim that network metadata
   is anonymous or outside privacy law.
3. **No raw mail identifiers:** RFC Message-ID, account addresses, folder names,
   UIDs, and other locators never leave a Brev client in cleartext.
4. **Provider authority is preserved:** the relay does not shadow or overwrite
   state a mail provider can authoritatively synchronize.
5. **Transport-neutral:** the same application-authenticated HTTPS API works
   behind Tailscale, Cloudflare Tunnel, Caddy, another reverse proxy, or a
   private LAN endpoint.
6. **Self-hosting first:** a user's PC, Mac, NAS, Raspberry Pi, or home server is
   sufficient. A VPS is optional for availability.
7. **Boring operations:** one small service, SQLite by default, bounded inputs,
   explicit migrations, backup/restore, and no mandatory control plane.

## Intended topology

```text
Brev on macOS / iPhone / iPad
  |-- local state and durable pending-operation queue
  |-- client-side identity resolution, encryption, and merge
  |
  `-- HTTPS through LAN / Tailscale / Tunnel / reverse proxy
        |
        `-- brev-syncd
              |-- device and workspace authorization metadata
              |-- opaque encrypted operation log
              `-- opaque encrypted client-created snapshots
```

If the host sleeps or goes offline, Brev continues locally and uploads pending
operations when the daemon is reachable again.

## Repository map

| Resource | Purpose |
| --- | --- |
| [`AGENTS.md`](AGENTS.md) | Canonical working contract for coding agents |
| [`docs/PLAN.md`](docs/PLAN.md) | Phased implementation plan and acceptance gates |
| [`docs/HANDOVER.md`](docs/HANDOVER.md) | Current state and exact next task |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Components, trust boundaries, storage, and flows |
| [`docs/PROTOCOL.md`](docs/PROTOCOL.md) | Draft v0 wire and device-lifecycle contract |
| [`docs/THREAT_MODEL.md`](docs/THREAT_MODEL.md) | Assets, threats, mitigations, and residual risks |
| [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) | Supported local and hosted deployment profiles |
| [`docs/RESOURCES.md`](docs/RESOURCES.md) | Required upstream standards and Brev context |
| [`CONTEXT.md`](CONTEXT.md) | Canonical domain language for protocol and product decisions |
| [`docs/agents/`](docs/agents/) | Issue-tracker, triage, and agent-workflow conventions |
| [`SECURITY.md`](SECURITY.md) | Private vulnerability reporting policy |
| [`WORKLOG.md`](WORKLOG.md) | Factual agent/session handoffs |

## Planned implementation shape

- Go daemon distributed as native binaries and a multi-architecture OCI image.
- SQLite storage with WAL, transactions, explicit schema migrations, and an
  encrypted-backup workflow.
- Versioned HTTP/JSON API carrying bounded opaque ciphertext envelopes.
- Ed25519 device authentication and signed membership records.
- Client-generated workspace secrets; the daemon never receives decryption
  keys.
- First delivery proves two simulated clients, then Brev Mac/iPhone/iPad with
  only pin and flag-colour state.

Algorithm choices in the protocol document are candidates until test vectors,
nonce rules, recovery behaviour, and an independent security review are
complete. Do not implement protocol crypto from this README.

## Non-goals for the first release

- Mail proxying, message fetch, search, indexing, or notifications.
- Credentials, OAuth, account provisioning, or provider configuration.
- Message bodies, subjects, correspondents, snippets, notes, drafts, or
  attachments.
- A browser administration dashboard.
- PostgreSQL, Redis, clustering, Kubernetes, or a hosted Brev control plane.
- Guaranteed instant background convergence while iOS/iPadOS is suspended.
- Arbitrary free-form synchronized records.

## License

`brev-syncd` is licensed under the GNU Affero General Public License v3.0. See
[`LICENSE`](LICENSE). The Brev application remains separately licensed under
MIT.
