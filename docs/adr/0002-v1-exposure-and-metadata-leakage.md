# ADR-0002: Keep v1 loopback-first and disclose metadata leakage

**Status:** accepted

Version 1 accepts ciphertext-size, request-timing, operation-volume, and
network-address leakage as documented transport metadata; it does not provide
padding, delayed upload, or traffic-analysis resistance guarantees. The daemon
binds to loopback. Any off-host reachability (LAN, tailnet, or public) is
provided only by a maintained TLS reverse proxy or private-network proxy such
as Tailscale Serve. The LAN-only profile is no exception: a LAN-facing reverse
proxy fronts the loopback daemon. V1 does not bind the daemon to a LAN
interface. V1 presents one owner and one Workspace per instance, while the
schema keeps Workspace isolation for a later multi-owner decision.

## Consequences

The product must not describe encrypted payloads as anonymous or uniformly
non-personal. Logs remain disabled by default, but the privacy disclosure must
enumerate metadata observable by a daemon, proxy, host, or network. A direct
public listener, a non-loopback daemon bind, traffic-shaping guarantees, and
multi-owner instances require separate decisions and review. Whether a
plaintext LAN-facing proxy is acceptable in v1 remains an open question; a
plaintext non-loopback daemon listener is not.
