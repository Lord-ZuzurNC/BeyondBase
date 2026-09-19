# ADR-0002 — Node to main server transport is mTLS over HTTP/2

- **Status:** Accepted
- **Date:** 2026-09-19
- **Relates to:** AC-SEC-01, AC-SEC-03, AC-SEC-10, AC-AUDIT-06, [ADR-0001](0001-relay-nodes-hold-no-state.md)

## Context

Every relay node talks to the main server for anything not in its public cache. That link carries player requests, so it must be encrypted (AC-SEC-01) and the main server must know which node is calling: to attribute traffic, to apply rate limits (AC-SEC-10), to log to the Server Log (AC-AUDIT-06) and to cut off a node that is compromised or retired.

Per-request player authentication is unchanged: the player's token is validated by the main server on every request (AC-SEC-03). This decision is only about the machine-to-machine channel underneath it.

## Decision

**Mutual TLS over HTTP/2.**

- The main server runs a **private certificate authority** for node certificates. It is not the public web PKI and issues nothing else.
- A node is **enrolled** by the operator in the console. Enrolment produces a node ID and a single-use enrolment token; the node generates its own key pair, never transmits the private key, and exchanges the token for its first certificate.
- Node certificates are **short-lived (30 days)** and renewed automatically over the same channel at half their lifetime. A node that fails to renew simply stops being served, which is the behaviour we want.
- The certificate's subject carries the **node ID**. The main server uses it for the per-node rate accounting, for Server Log entries and for the node's lane in the console.
- **Revocation is a list in the database**, checked in process on every connection. No OCSP and no external service. Revoking a node in the console takes effect on its next connection, and existing connections are dropped.
- Player-facing TLS at the node is ordinary public TLS with Let's Encrypt certificates, obtained by the node itself.
- **The forwarded client IP is trusted only when it arrives on an authenticated node connection.** Rate limits for unauthenticated requests are counted per IP (AC-SEC-10), so an unverified forwarding header would hand anyone a way to defeat them.
- HTTP/2 carries both ordinary requests and proxied WebSocket connections, so there is one protocol, one port and one certificate story.

## Consequences

**Good**
- Encryption and machine identity come from the same mechanism, with nothing extra to run.
- A node can be revoked instantly, from the console, per node.
- Short-lived certificates make a stolen key a limited problem without needing a revocation to be noticed first.
- It works over any network, including the public internet, so nodes can be hosted anywhere.

**Bad, and accepted**
- We operate a small CA: issuance, renewal, revocation and the backup of its key. That is genuine work, and the CA key is now one of the most sensitive secrets in the system.
- Certificate expiry is a new class of outage. The console must show certificate age per node as a lane, with an OUT state well before expiry rather than on it.
- Clock skew breaks mTLS. Nodes need working NTP.

## Alternatives considered

- **WireGuard tunnel, plain HTTP inside.** Simpler application code and strong crypto, but the tunnel becomes a second thing to operate and debug, and peer management ends up outside the console the operator actually uses. Keeping identity inside the application is what makes per-node revocation, rate accounting and the console's node lanes possible without a second control plane. Still the right escape hatch if the network between a node and the main site is ever untrusted at a level TLS alone should not face.
- **TLS with a shared bearer token per node.** Easy, but one leaked token is as good as all of them, rotation means touching every node, and the token ends up in logs and environment dumps. Rejected.
- **gRPC or QUIC.** Fine transports, but they add a schema and a dependency without solving anything this decision is about. The choice of HTTP/2 does not preclude them later for a specific hot path.
