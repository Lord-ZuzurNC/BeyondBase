# ADR-0001 — Relay nodes hold no state

- **Status:** Accepted
- **Date:** 2026-09-19
- **Relates to:** AC-SEC-06, AC-SEC-07, AC-SEC-08, AC-DEV-05, AC-RT-01, TCG_Beyond TODO ("nodes … gather and transit data from Main Server to players")

## Context

TCG_Beyond needs to spread player traffic across several machines. The TODO describes "nodes" that run the stack only to carry data between the main server and players.

Three readings were possible: a stateless relay, a read replica holding a local copy of the database, or a full peer that can serve writes. The platform's hard rules make that choice consequential. The server is the only authority for rewards, ownership, card values, booster results, random outcomes and battle results (AC-SEC-06, AC-SEC-07). Museum changes are atomic (AC-SEC-08). The Economy Ledger is immutable and append-only. Events are idempotent by UID (AC-DEV-05). Any design where an edge machine can accept a write, or answer from a copy that may lag, puts those guarantees at risk of being violated quietly, which is the one failure mode this platform exists to prevent.

## Decision

**A relay node holds no authoritative state.** It does three things:

1. Terminates player TLS connections.
2. Serves cached **public, immutable** data: card definitions, card images and other static assets, frontend bundles and i18n bundles.
3. Proxies everything else to the main server over the channel in [ADR-0002](0002-node-to-main-transport.md).

A node never writes to the database, never decides a game outcome, never holds a session or a token beyond the life of the request it is proxying, and never stores player data on disk.

**Cache coherence is done with generation stamps, not an invalidation protocol.** Every cacheable class of data carries a counter held by the main server: the card database sync generation, the configuration version (GS-DEV-01), the season ID, the asset build ID. Cache keys include the stamp. The main server exposes a tiny endpoint returning the current stamps, which nodes poll every 10 seconds. Bumping a counter makes every old key unreachable at once. There is no purge fan-out, no message bus and nothing to get stuck.

**Horizontal scaling of the application happens at the main site, not at the edge.** Several stateless application instances sit behind a load balancer and share one PostgreSQL. "Instances in sync" means they hold no local state worth syncing: the database is the only shared truth.

**Explicitly never cached at a node:** player state, Coiniverse balances, museum contents, market listings and bids, joute state, chat, notifications, sessions, tokens, and anything from the ledger or the audit domains.

**WebSocket traffic** (market bids, AC-RT-01) is proxied connection for connection. The node adds no logic.

<!-- ponytail: one upstream WS connection per proxied player. If a popular listing makes that the bottleneck, multiplex one upstream subscription per listing at the node and fan out locally. -->

## Consequences

**Good**
- No invariant can be violated at the edge, because no edge machine can mutate anything.
- No replication lag and therefore no stale-read class of bug.
- A compromised node exposes only public card data. It holds no personal data, which also keeps it out of scope for the GDPR retention rules in AC-AUDIT-05.
- Nodes need no backup and no migration. Losing one means losing capacity, never data.
- A node is cheap to run and can be added or removed at any time.

**Bad, and accepted**
- A player far from the main server still pays the round trip for anything dynamic. Nodes relieve bandwidth and connection load, not distance. If latency in a distant region ever becomes a real complaint, that is the moment to revisit read replicas, with the stale-read risk understood.
- The main site is the single point of failure. This matches the current reality of one operator and one owned server, and it keeps the ledger unambiguous.

## Alternatives considered

- **Read replica per node.** Faster distant reads, at the price of replication lag, stale-read bugs and personal data sitting on every edge machine. Rejected: the cost lands on correctness and on GDPR scope, for a benefit no measurement has asked for yet.
- **Full peer instances.** Needs distributed consensus and conflict resolution over exactly the data that must never be resolved by a heuristic: balances, ownership and the ledger. Rejected as disproportionate to a game run by one operator.
