# ADR-0003 — Console operators are a separate identity, authenticated with passkeys

- **Status:** Accepted
- **Date:** 2026-09-19
- **Relates to:** GS-AUTH-06, GS-AUTH-12, GS-AUTH-15, AC-ADMIN-01, AC-ADMIN-03, AC-AUDIT-03, AC-SEC-02

## Context

The BeyondBase console operates the platform: schema and data, nodes, jobs, TCG ingestion, configuration, and reading the ledger and audit logs. Its users are the Owner and the Developer role only. The other three admin roles (Super Admin, Support, Moderator) work in TCG_Beyond's in-game `/admin` panel and never touch this console.

Two identities were candidates: a TCG_Beyond player account carrying an admin role (GS-AUTH-06), or an identity of its own. The long-term infrastructure plan also names an IAM with OIDC, MFA and SSO, so whatever is built now has to be replaceable without rewriting the console or losing audit history.

## Decision

**A separate operator identity, with passkeys, behind an interface that OIDC can take over.**

- Operators live in their **own table**, unrelated to player accounts. An operator is not a player, and a player account cannot grant console access. Console roles are **Owner** and **Developer** only.
- Authentication is **WebAuthn passkeys**: discoverable credentials with user verification required. There is no password, no email code and no password reset flow, so there is nothing to phish, leak or reset over email.
- **Two passkeys minimum per operator** (typically the desktop and the phone), enforced at enrolment. A single authenticator is a lockout waiting to happen.
- One **single-use recovery code** is issued at enrolment and stored only as an Argon2 hash. Using it is an Admin Action Log event and forces the enrolment of a new passkey immediately.
- **Session:** a host-only cookie, `Secure`, `HttpOnly`, `SameSite=Strict`, idle timeout 30 minutes to match GS-AUTH-12. There is no refresh token, because re-authenticating is one touch. "Sign out everywhere" invalidates every session for that operator, as GS-AUTH-15 does for players.
- **Network:** console routes are served on a **separate listener** bound to the private network, never on the public one. Authentication and network reachability are independent barriers, and neither is asked to carry the other.
- **Audit:** every state-changing console action writes an Admin Action Log entry carrying the operator ID (AC-ADMIN-03, AC-AUDIT-03). Reading the ledger or a log is itself recorded, since the console is the only place those can be read.
- **The OIDC seam:** identity resolution sits behind a single interface that returns an operator ID and a role. The passkey store is its first implementation; an OIDC provider becomes a second one, mapping a claim to a role. Operator IDs are internal UUIDs that never change, so a later migration keeps every audit record pointing at the same person.

## Consequences

**Good**
- No password anywhere in the operator path, and no email dependency for access.
- Console access and player accounts cannot be confused, and an Owner losing a player account does not lose the platform.
- The private-network listener means an internet-facing bug in the game's routes cannot expose the console.
- The seam makes Authelia, FreeIPA or another OIDC provider a later addition rather than a rebuild.

**Bad, and accepted**
- Passkeys need a browser and an authenticator that support them. On the phone this is expected to be the platform authenticator, which ties console access to that device's own lock screen.
- We implement WebAuthn registration and assertion ourselves, including the relying-party ID, which is tied to the console's hostname. Changing that hostname later invalidates existing passkeys and means re-enrolment.
- Recovery rests on a single code plus a second passkey. If both are lost, recovery is manual database work by the Owner. That is an accepted consequence of having no email path.

## Alternatives considered

- **Reuse the player account with an admin role.** One identity, but it ties console access to the player login surface and its email flows, and it means a phishable password can reach the ledger. Rejected.
- **External IdP from day one.** Matches the long-term plan, but it makes the console unreachable when that service is down and adds a service to run before there is a second person to run it for. Deferred behind the seam above.
