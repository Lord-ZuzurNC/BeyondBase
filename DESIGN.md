---
name: BeyondBase
description: The operator console of TCG_Beyond's platform, drawn as a normalling schedule on black glass.
---

<!-- SEED: established with the user before implementation; re-run /impeccable document once there's code to capture the actual tokens and components. -->

# Design System: BeyondBase

## Overview

**Creative North Star: "The Normalling Schedule"**

BeyondBase's console is the backlit schedule taped inside a machine-room door. Every thing the platform runs is a **numbered lane**: the main server, each relay node, each instance, each scheduled job, each TCG ingestion plugin, each configuration key and each failure counter. Every lane is tied to its *normal* by a link line: its expected state, its default value, its healthy reading. The console exists to answer one question fast: *what is out of normal, and did someone break it on purpose?*

The world is unlit black glass with one signal amber. The console is dense and ruled, with hierarchy built only from scale and inversion. It does not use cards, gradients, glass blur or decorative charts. State is drawn in the **stroke of the link rail**, never in a hue, which suits both a phone glanced at while on call and the product's rule that status is never carried by colour alone. History (the Economy Ledger, the audit domains) appears as a sealed schedule: it can be read and exported, and nothing in it can be edited.

This is its own world. It deliberately does not share TCG_Beyond's Catppuccin identity.

**Key Characteristics:**
- Numbered lanes are the one structural unit, and lane numbers are durable anchors you can link to and re-enter.
- The link rail carries state: its stroke pattern shows the state, and a text label always sits beside it.
- One signal amber on black glass. Foreign colour appears only inside framed images such as TCG card art.
- One amber inversion plate per screen marks the single dominant fact.
- Nothing spans the full width, and dark gutters hold both edges.

## Colors

A single-ink system: black glass for the ground, and signal amber for every mark, in stepped intensities. Strategy: **Restrained**, taken to its limit.

### Primary
- **Signal Amber** (#FFB000): every mark that means something, including text, legends, live link rails, lane numbers, controls and the inversion plate. On Black Glass it measures about 10.8:1.

### Neutral
- **Black Glass** (#0A0A0A): the ground of every surface. The console is dark only, because it is used at a desk late at night and on a phone while on call.
- **Dim Amber** (#7A5200): structural hairlines, lane dividers and ruling. At about 2.9:1 it is **below 3:1**, so it never carries state or marks a control's boundary.
- **Inactive Amber** (#3A2600): disabled fills and the hollow wells of empty lanes. Decorative only.
- **Amber Halftone** (#FFB000 at 20%) and **Amber Grid** (#FFB000 at 40%): etched-glass textures behind the lane grid and inside empty wells. Never behind text.

### Named Rules
**The One Ink Rule.** Amber is the only ink. The console has no red for errors and no green for healthy: those are stroke patterns plus words. Foreign colour enters only inside a framed image.

**The Three-to-One Rule.** Anything that carries state or marks the edge of a control uses Signal Amber. Dim and Inactive Amber are structure and decoration, never meaning.

## Typography

**Display and Label Font:** **Barlow Condensed** (SIL OFL), self-hosted. The console runs on a private network, so fonts are served from the binary and never from a CDN.
**Mono, Prose and Data Font:** **Mononoki** (SIL OFL, <https://madmalik.github.io/mononoki/>), self-hosted. It carries prose, notes, every number and every raw UID, hash or payload.

**Character:** a condensed grotesque for naming things, and a screen monospace for reading them. Barlow Condensed names the lane; Mononoki carries everything measured or quoted, the way the schedule on a machine-room door prints its labels wide and its readings fixed-width. Hierarchy comes only from scale and inversion, never from colour or a third family.

### Hierarchy
- **Display / Rank** (Barlow Condensed, tightly set, very large): only the one inversion plate per screen, for example "02 OUT".
- **Lane number** (Mononoki, large): the anchor of every row. Monospaced by nature, so lane numbers align without a feature flag.
- **Item** (Barlow Condensed, caps): the lane's name, for example MAIN, RELAY-03, TCG SYNC · SCRYFALL.
- **Legend** (Barlow Condensed, small tracked caps): column heads (LANE · ITEM · RAIL · STATE · NORMAL) and section names.
- **State label** (Barlow Condensed, small caps): NORMAL, OUT, HELD, DISABLED, ISOLATED, beside the rail.
- **Data** (Mononoki): counts, latencies, Coiniverse amounts, UTC timestamps, versions, UIDs and hashes.
- **Prose** (Mononoki, sentence case): notes, failure reasons and help, at most 65ch.

### Named Rules
**The Tabular Rule.** Every number (lane numbers, counts, Coiniverse amounts, latencies, UTC times, versions) is set in Mononoki, whose figures are monospaced by construction. Where a number must sit in Barlow Condensed, it carries `font-variant-numeric: tabular-nums`.

**The Two Voices Rule.** Barlow Condensed names, Mononoki measures. A label is never set in the mono, and a reading is never set in the condensed. There is no third family.

**The Inversion Rule.** Rank is shown by knocking type dark out of a solid amber plate. There is one plate per screen, and nothing else is inverted.

## Layout

- **The lane grid is the page.** Rows are numbered lanes, grouped into sections (MAIN, RELAYS, INSTANCES, JOBS, PLUGINS, COUNTERS, CONFIG). The columns never move: **LANE · ITEM · RAIL · STATE · NORMAL**, with an optional note line under an out-of-normal lane.
- **A fixed left legend strip** holds the product mark, the system-status block (total / NORMAL / OUT / HELD), the UTC clock and navigation. The lane grid scrolls beside it.
- **Dark gutters** hold both edges at every width. No element runs edge to edge.
- **Density is high on purpose.** Lanes are compact and ruled, and a long schedule collapses with an ellipsis lane (`…`) between runs of NORMAL lanes. OUT, HELD and ISOLATED lanes always stay visible.
- **Phone:** the legend strip becomes a bottom bar. Lanes stay as a single list of LANE · ITEM · STATE, and narrower widths drop columns (NORMAL first, then RAIL), never the gutters and never the STATE label.
- [The spacing scale and breakpoints will be set during implementation.]

### Named Rules
**The Drift Rule.** The lane grid drifts one hairline per second under static type: the ground moves, nothing else does. It is on by default and stops entirely under `prefers-reduced-motion: reduce`. Type, rails, lane numbers and state labels never move, and no drift ever changes what a lane reads.

## Elevation & Depth

Flat. The system uses no shadows, no blur and no raised surfaces. Depth comes from etched texture (halftone and grid behind the lane field) and from ruling. Selection is shown by doubling the rail, not by lifting anything.

**The Glass Stays Flat Rule.** Nothing floats. Dialogs and drawers are framed panels ruled onto the same glass, set off by an amber hairline frame, not by a shadow.

## Shapes

Hairline frames and square or barely softened corners. The only curves in the system are the link rail's round terminal node and the open ring. Lanes, panels and buttons are rectangular, framed in hairline, and snapped to the lane grid.

### The Link Rail (the signature form)
Every lane owns a rail that runs from its ITEM to its NORMAL terminal. The stroke pattern is the state:

| Rail stroke | State | Meaning in BeyondBase |
|---|---|---|
| unbroken line → round terminal | **NORMAL** | the node is live, the job succeeded, the value equals its default |
| line broken by `//` | **OUT** | the node is lagging or down, a sync failed, a counter spiked, a config value differs from its default |
| single cross-tick | **HELD** | a job is paused, a node is draining, a plugin is suspended |
| clean gap | **DISABLED** | out of service by design |
| doubled line | **SELECTED** | the lane is the current selection or focus |
| ends in an open ring | **DESTRUCTIVE** | the action on this lane cannot be undone (forget node, revoke all sessions, purge) |
| dashed segment ending in ⊗ | **ISOLATED** | an operator broke the normal on purpose. A note line carries the reason, the operator ID and the UTC time |

Every rail state also shows its word in the STATE column. A broken rail **never animates closed**: returning to NORMAL is a new event that gets recorded, not a transition.

## Do's and Don'ts

### Do:
- **Do** model every monitored or configurable thing as a numbered lane with a rail to its normal.
- **Do** show state with the rail stroke **and** its word (NORMAL, OUT, HELD, DISABLED, ISOLATED), every time.
- **Do** give every ISOLATED or OUT lane its reason line: the failure reason, or the operator note with the operator ID and UTC time.
- **Do** use CSS custom properties for every colour, and never hard-code a colour in a component rule.
- **Do** use tabular figures for all numbers and UTC for all timestamps.
- **Do** frame foreign imagery (TCG card art during ingestion review) in an amber hairline with a filled amber caption bar.
- **Do** render the ledger and the audit logs read-only: rails end in a sealed terminal, with no edit or delete affordance anywhere.

### Don't:
- **Don't** use hue to carry state. No red, no green, no status-coloured chips.
- **Don't** use Dim or Inactive Amber for anything that carries meaning or marks a control's boundary.
- **Don't** add metric cards, gradient charts, glass blur, glow or shadows. That is the category default this world refuses.
- **Don't** put more than one inversion plate on a screen.
- **Don't** let any element span the full width, or drop the edge gutters at narrow widths.
- **Don't** animate a broken rail back to unbroken. Returning to NORMAL is a recorded event, not a transition.
- **Don't** let anything but the lane-grid ground move, and stop even that under `prefers-reduced-motion`.
- **Don't** load a font from a CDN. Barlow Condensed and Mononoki ship with the binary.
