# Required resources

An agent should read these sources before finalizing the relevant design. Prefer
the standards and official documentation over secondary summaries.

## Brev context

- [Brev repository](https://github.com/henrikogaard/brev)
- [ADR-0008: telemetry and privacy](https://github.com/henrikogaard/brev/blob/main/ADRs/0008-telemetry-and-privacy.md)
- [ADR-0031: message flag colours and provider-agnostic tagging](https://github.com/henrikogaard/brev/blob/main/ADRs/0031-message-flag-colors-and-provider-agnostic-tagging.md)
- [ADR-0043: standards-first IMAP/SMTP roadmap](https://github.com/henrikogaard/brev/blob/main/ADRs/0043-standards-first-imap-smtp-roadmap.md)
- [ADR-0044: IMAP/SMTP backend foundation](https://github.com/henrikogaard/brev/blob/main/ADRs/0044-imap-smtp-backend-foundation.md)
- [ADR-0045: full IMAP sync/cache engine](https://github.com/henrikogaard/brev/blob/main/ADRs/0045-full-imap-sync-cache-engine.md)
- [ADR-0060: provider-backed workflow state](https://github.com/henrikogaard/brev/blob/main/ADRs/0060-provider-backed-workflow-state.md)
- [Current `MessageHeader` identity notes](https://github.com/henrikogaard/brev/blob/main/packages/BrevBackend/Sources/BrevBackend/Models.swift)

Do not copy stale Brev state into this repository. Verify current `main` and the
relevant project/issue state before integrating.

## Mail identity standards

- [RFC 9051: IMAP4rev2](https://www.rfc-editor.org/rfc/rfc9051.html)
- [RFC 8474: IMAP Object Identifiers](https://www.rfc-editor.org/rfc/rfc8474.html)
- [RFC 5322: Internet Message Format](https://www.rfc-editor.org/rfc/rfc5322.html)
- [RFC 8621: JMAP Mail](https://www.rfc-editor.org/rfc/rfc8621.html)

Important: IMAP UID identity is mailbox + UIDVALIDITY + UID scoped. RFC
Message-ID can be absent or duplicated. RFC 8474 object IDs are optional. Test
the capability and failure matrix rather than assuming one universal ID.

## Cryptographic standards

- [RFC 8032: EdDSA](https://www.rfc-editor.org/rfc/rfc8032.html)
- [RFC 7748: X25519/X448](https://www.rfc-editor.org/rfc/rfc7748.html)
- [RFC 5869: HKDF](https://www.rfc-editor.org/rfc/rfc5869.html)
- [RFC 8439: ChaCha20-Poly1305](https://www.rfc-editor.org/rfc/rfc8439.html)
- [NIST SP 800-38D: GCM](https://csrc.nist.gov/pubs/sp/800/38/d/final)
- [Apple CryptoKit](https://developer.apple.com/documentation/cryptokit)
- [Go cryptography packages](https://pkg.go.dev/golang.org/x/crypto)

Using a standard primitive does not automatically make a safe protocol. Freeze
canonical bytes, associated data, key separation, nonce/counter persistence,
recovery, and test vectors before implementation.

## Storage and service hardening

- [SQLite WAL](https://sqlite.org/wal.html)
- [Go `net/http`](https://pkg.go.dev/net/http)
- [Go fuzzing](https://go.dev/doc/security/fuzz/)
- [Go vulnerability management](https://go.dev/doc/security/vuln/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/)
- [SLSA](https://slsa.dev/)

## Deployment transports

- [Tailscale Serve](https://tailscale.com/docs/reference/tailscale-cli/serve)
- [Tailscale VPN On Demand](https://tailscale.com/docs/features/client/ios-vpn-on-demand)
- [Cloudflare Tunnel published applications](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/routing-to-tunnel/)
- [Caddy automatic HTTPS](https://caddyserver.com/docs/automatic-https)
- [Apple background task strategy](https://developer.apple.com/documentation/backgroundtasks/choosing-background-strategies-for-your-app)
- [Apple background push limitations](https://developer.apple.com/documentation/usernotifications/pushing-background-updates-to-your-app)

## Licensing

- [GNU AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html)
- [Open Source Definition](https://opensource.org/osd)

Every dependency must be compatible with AGPL distribution. Record exact
versions and license files in a future `THIRD_PARTY_LICENSES.md` before the first
release.
