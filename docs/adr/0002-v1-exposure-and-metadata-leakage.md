# ADR-0002: Keep v1 loopback-first and disclose metadata leakage

**Status:** accepted

Version 1 accepts ciphertext-size, request-timing, operation-volume, and
network-address leakage as documented transport metadata; it does not provide
padding, delayed upload, or traffic-analysis resistance guarantees. The daemon
binds to loopback and public reachability is provided only by a maintained TLS
reverse proxy or private-network proxy such as Tailscale Serve. V1 presents one
owner and one Workspace per instance, while the schema keeps Workspace
isolation for a later multi-owner decision.

## Consequences

The product must not describe encrypted payloads as anonymous or uniformly
non-personal. Logs remain disabled by default, but the privacy disclosure must
enumerate metadata observable by a daemon, proxy, host, or network. A direct
public listener, traffic-shaping guarantees, and multi-owner instances require
separate decisions and review.
