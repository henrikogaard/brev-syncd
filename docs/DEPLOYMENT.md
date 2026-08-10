# Deployment profiles

This document describes planned support. Commands are illustrative until a
released binary and container exist.

## Core rule

`brev-syncd` does not require cloud hosting. It can run on a user's desktop PC,
Mac, Windows machine, Linux host, NAS, Raspberry Pi, or home server.

The host must be awake and reachable for clients to exchange operations. If it
is offline, clients keep working locally and synchronize later.

## Availability profiles

| Profile | Data location | Reachability | Expected behaviour |
| --- | --- | --- | --- |
| LAN only | User's computer | Same network | Maximum locality; no away-from-home sync |
| Tailscale Serve | User's computer | Private tailnet | Recommended home setup |
| Cloudflare Tunnel | User's computer | Public hostname through Cloudflare | No inbound port forwarding; Cloudflare sees transport metadata |
| Direct HTTPS | User's computer | Public DNS/IP through user's proxy | Maximum operator control and responsibility |
| VPS | Encrypted database on rented host | Usually 24/7 | Optional availability choice |

Cloud hosting changes availability and operational responsibility, not the
payload-encryption boundary.

## Planned process defaults

- Listen on `127.0.0.1:<port>` by default.
- One explicit data directory containing the SQLite database and local
  instance-only secrets/capabilities.
- Run as an unprivileged service account.
- No outbound network requirement during normal serving.
- No access log by default.
- `/healthz` returns only liveness/readiness and no workspace/device counts.
- Graceful shutdown completes or rolls back current transactions.
- Database migrations run under a lock and fail without partially advancing.

V1 binds the daemon to loopback. Any reachable LAN, tailnet, or public
transport is terminated by a maintained TLS or private-network proxy. V1 does
not implement a direct public daemon listener.

## LAN-only

Suitable when all devices synchronize at home.

Planned characteristics:

- TLS is still recommended; application signatures and payload encryption are
  mandatory regardless.
- Bonjour/mDNS may advertise setup discovery without workspace identity or
  sensitive status.
- The user must handle local firewall access.
- No phone/tablet sync occurs away from the LAN.

LAN discovery is convenience only and must never auto-enrol a device.

## Tailscale Serve — recommended home profile

Planned shape:

```text
brev-syncd -> http://127.0.0.1:8484
tailscale serve -> HTTPS tailnet hostname -> localhost:8484
```

Benefits:

- no router port forwarding;
- tailnet-only reachability;
- HTTPS termination and stable MagicDNS hostname;
- suitable for PC, NAS, home server, and VPS hosts.

Constraints:

- every Brev device needs tailnet access;
- iOS/iPadOS may depend on VPN On Demand for transparent background reachability;
- only one iOS/macOS VPN application may own On Demand at a time;
- Tailscale is defence in depth, not Brev device authorization.

## Cloudflare Tunnel

Planned shape:

```text
brev-syncd -> http://127.0.0.1:8484
cloudflared -> https://sync.example.net -> localhost:8484
```

Benefits:

- no inbound port or stable home IP required;
- phone/tablet need no VPN;
- public hostname works through ordinary HTTPS clients.

Constraints:

- Cloudflare carries the traffic and observes normal transport metadata;
- public endpoints need application rate/concurrency/storage limits;
- do not embed one shared Cloudflare service-token secret in all Brev clients;
- Brev device authorization remains mandatory.

The encrypted payload prevents a TLS-terminating proxy from reading domain
metadata, assuming the client protocol is correctly implemented.

## Public HTTPS through a reverse proxy

Advanced users may expose the loopback daemon through Caddy, nginx, Traefik,
HAProxy, or another maintained TLS reverse proxy. The daemon must not be
forwarded directly to the Internet.

Required documentation before support:

- automatic certificate issuance and renewal;
- HTTP/2 and timeout behaviour;
- request/body limits at both layers;
- trusted-proxy header policy;
- firewall and router configuration;
- dynamic DNS/CGNAT limitations;
- daemon application authentication;
- update, backup, and incident response.

V1 does not support a direct daemon TLS listener. Supporting one later requires
a separate architecture and public-exposure review.

## VPS

A small VPS is optional for users who want 24/7 availability without an
always-on home device. The intended resource budget, to be verified by Phase 4,
is one low-end CPU, less than 256 MiB steady-state daemon memory, and storage
proportional to opaque operations/snapshots rather than mailbox size.

These are design targets, not current benchmarks.

The VPS operator should still use an unprivileged process/container, encrypted
host volume where available, TLS, firewalling, automatic security updates,
backups, and Brev application authentication.

## Planned packaging

- Native archives with checksums for Linux amd64/arm64, macOS arm64/amd64, and
  Windows amd64.
- Multi-architecture OCI image running non-root with one writable volume.
- Docker Compose example with a health check and no default public bind.
- systemd hardened unit.
- launchd user/system service guidance.
- Windows service guidance.
- SBOM and signed release provenance.

The container must not require privileged mode, host networking, Docker socket,
or broad filesystem mounts.

## Backup and migration

Planned operational contract:

- `backup` produces a consistent opaque database archive plus format/version
  metadata.
- `restore` validates archive structure and refuses unsafe downgrade.
- Backups do not contain workspace decryption keys.
- An authorized client can reseed a new empty daemon from an encrypted snapshot
  even when the old daemon is unavailable.
- Recovery documentation distinguishes host recovery from workspace-key
  recovery.

## Mobile behaviour

- Foreground sync should converge within seconds when the daemon is reachable.
- iOS/iPadOS background refresh is opportunistic and may be delayed.
- A sleeping host and a suspended app compound delay; neither is data loss.
- The app retries on next foreground activation and exposes pending/unreachable
  state.
- An optional opaque APNs wake relay is deferred and would be a separate privacy
  decision.

## Deployment acceptance matrix

Every supported profile eventually needs evidence for:

- clean install;
- first-device bootstrap and QR pairing;
- Mac/iPhone/iPad connectivity;
- restart and host reboot;
- offline queue and later convergence;
- upgrade and schema migration;
- revoked-device rejection;
- backup/restore and client reseed;
- logs/database free of prohibited plaintext;
- documented uninstall and data removal.
