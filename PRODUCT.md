# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

- **Server:** Rust, PostgreSQL. A single self-hosted server binary.
- **Console:** Leptos (Rust compiled to WASM), served by the server binary.
- Everything else (web framework, migrations, queue, cache, transport between nodes) is still open. It gets decided in ADRs, with TCG_Beyond's priorities in mind: modern, low-resource, high-capacity, FOSS, and a bonus for EU-made or EU-hosted.

## Users

- **Primary:** the Owner (the person who runs TCG_Beyond, operates the servers and writes the code) and the **Developer** admin role. They use the console to operate the platform: manage data and schema, check on nodes and instances, run and monitor jobs and TCG ingestion, look at events, the ledger and audit logs, and change configuration.
- **Not users of the console:** Super Admin, Support and Moderator. Game operations such as events, council, seasons, players, moderation, appeals and announcements stay in TCG_Beyond's in-game `/admin` panel (AC-ADMIN-*). BeyondBase supplies the data and authorization behind that panel. It does not replace its UI.
- **Indirect consumers:** the TCG_Beyond game server and frontend, which use BeyondBase's APIs.

## Product Purpose

BeyondBase is a custom Backend-as-a-Service built only for TCG_Beyond, a web game where players open boosters of cards drawn from every TCG. It fills the role PocketBase or Supabase would fill, with the game's non-negotiable invariants enforced by the platform itself, so the game code cannot accidentally break them.

Success means:
- TCG_Beyond's game logic never has to re-implement auth, idempotency, the ledger, audit or config.
- A violation of an invariant is impossible, or at least rejected and visible, instead of being silent.
- One operator can run the whole system, including its nodes, from the console on a desktop, or check on it from a phone.

## Positioning

Unlike a general-purpose BaaS, BeyondBase treats TCG_Beyond's rules as platform guarantees:

- **Game invariants built in:** an append-only, immutable Economy Ledger (AC-ECO-*). Idempotent events with unique UIDs, where a duplicate is logged as `SKIPPED — already processed` (AC-DEV-05). Integer-only Coiniverse with truncation, and no floating point (AC-DEV-03). Append-only audit domains (AC-AUDIT-*). A configuration layer with versioned formulas (GS-DEV-01, AC-DEV-04).
- **Node topology is native:** a main server that holds all data, relay nodes that carry traffic between the main server and players to spread the load, and several synchronized instances. All of these are first-class concepts, and every link is encrypted.
- **TCG plugin ingestion:** each TCG data source (Scryfall, TCGdex, YGOProDeck, the FaB cards repo, Lorcana API and others) is an independent plugin (AC-DEV-02). Adding a TCG never touches the core ingestion code. Sync status and history are platform features.
- **Lean and sovereign:** self-hosted, FOSS (GPLv3), low-resource, with no SaaS dependency.

## Operating Context

- The console is reachable **only over a private network**, never publicly.
- It is used on a **desktop** for daily operation, and on a **phone** for on-call checks such as a node going down, a sync failing, webhook failures or error spikes.
- Scheduled work it oversees: the monthly TCG resync, ledger file rotation on the 1st of each month at 00:00 UTC (AC-ECO-05), log rotation, and season snapshots.
- Things it must make observable (from TCG_Beyond's TODO): app and database metrics, API latency, error rate, queue depth, failures in booster generation, payment webhooks, market transactions, joutes and external TCG APIs.

## Capabilities and Constraints

- **Auth, sessions, roles:** registration with an email code, 30-minute access tokens, single-use rotating refresh tokens, concurrent sessions, log out from all devices, session invalidation when the password changes (GS-AUTH-*). Hashing is adaptive (Argon2/bcrypt). Email and subscription status are encrypted at rest. No payment method data is ever stored. There are 5 admin roles plus the access matrix in AC-AUDIT-07.
- **Data, events, ledger, audit:** collections and records, idempotent events, the Economy Ledger (a new file each month, kept permanently, immutable even for the Owner), 4 audit domains plus the Server Log, and the config layer. Log retention is 3 months, or 1 month when an entry holds personal data (AC-AUDIT-05).
- **Realtime, jobs, plugins:** WebSocket for market bids (AC-RT-01), polling for joute, chat and notifications, scheduled jobs, TCG ingestion plugins, and API rate limits per endpoint category (AC-SEC-10).
- **Nodes and multi-instance:** enrolling nodes, issuing and revoking their certificates, watching their health. Relay nodes hold no authoritative state and only cache public data (ADR-0001); the link to the main server is mTLS over HTTP/2 (ADR-0002). Application instances are stateless and share one PostgreSQL.
- **Console access:** operators are a separate identity from players, authenticated with WebAuthn passkeys, on a listener bound to the private network, with an OIDC seam for later (ADR-0003).
- **Hard rules:** the client is never trusted (AC-SEC-06/07). Changes to the Museum and the economy are atomic. Historical data is never destroyed. No secret, token or personal data is ever logged in plaintext.
- **Authority:** TCG_Beyond's `docs/requirements/` (GS-*, GR-*, AC-*) is the product truth. BeyondBase implements the platform side of those IDs and never reinterprets them.
- **Decided in `docs/decisions/`:** node topology and state (ADR-0001), node-to-main transport (ADR-0002), console operator identity (ADR-0003).
- **Still open:** the job runner and scheduler, the metrics and observability stack, the backup and disaster-recovery approach, and the remaining stack picks (web framework, migrations, cache).

## Brand Commitments

- The name is **BeyondBase**, the "Base" underneath TCG **Beyond**.
- The README carries the KeepAndroidOpen caution banner, the same as TCG_Beyond.

## Evidence on Hand

- TCG_Beyond requirements: `~/Lab/TCG_Beyond/docs/requirements/{game-system,game-rules,architecture}.md`.
- No code, metrics, users or benchmarks exist yet. Never invent performance figures, adoption numbers or comparisons.

## Product Principles

1. **Invariants belong to the platform.** Anything that must never happen (a double-applied event, a mutated ledger row, a float in Coiniverse) is rejected by BeyondBase, not by convention.
2. **Operate, not administer the game.** The console is for running the platform. Game decisions stay in TCG_Beyond's `/admin`.
3. **Make failure loud.** Failed syncs, skipped duplicates, lagging nodes and failing webhooks show up without anyone having to dig for them.
4. **Read-only by default for history.** The ledger and audit logs can be viewed and exported. Nothing in the console can edit or delete them.
5. **One operator can run it.** Low resource use, a single binary, few moving parts.

## Accessibility & Inclusion

- WCAG 2.2 AA, fully usable by keyboard, and usable at phone width for on-call checks.
- Status is never shown by colour alone: health, severity and failure states always carry text or an icon as well.
