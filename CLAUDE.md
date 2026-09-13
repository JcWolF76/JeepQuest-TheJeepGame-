# CLAUDE.md — Conventions for Claude Code sessions

> Read this file at the start of every session before making changes.

---

## 1. Ship to `main`

GitHub Pages serves JeepQuest from `main` at the repo root:
**https://jcwolf76.github.io/JeepQuest-TheJeepGame-/**

Same deploy model as PlateQuest: Pages builds from `main` / (root), no
Actions workflow, rebuild takes ~1 minute after a push. Per-session
feature branches (`claude/<slug>-<id>`) exist so concurrent sessions
don't collide — not because `main` is off-limits. Default workflow for
shippable work: commit on the session branch, merge into `main`, push
`main`, tell the user it's live.

Skip the merge-to-main step only if the user says the work is WIP or
asks for a PR-review flow on that task.

## 2. Project identity

Same three-layer identity as PlateQuest — see that repo's CLAUDE.md for
the full rationale. Short version:

- **Person:** Jesse Bliss — copyright holder, appears on LICENSE.
- **Studio:** Sparkasia Studios — brand/publisher identity, user-facing.
- **Handle:** JcWolF — canonical dev/player handle (capital F, rest
  lowercase). Used in credits, bylines, player tag.
- **Handle (GitHub variant):** JcWolF76 — GitHub username only, never
  used as a brand handle in credits.

Never write `JcWolf`, `JcWoLF`, `JCWOLF`, or other casing variants.

## 3. Sister titles

JeepQuest is published by Sparkasia Studios alongside PlateQuest and
AnimalQuest. Each title is its own repo. Don't reference unreleased
Sparkasia titles or roadmap plans in user-facing copy (splash text,
changelog, README) — that's internal information.

## 4. License

Proprietary, not open source — see `LICENSE`. New source files get the
header:

```
JeepQuest (The Jeep Game) © 2026 Jesse Bliss (JcWolF). All rights reserved.
Proprietary software — see LICENSE.
```

## 5. Layout

```
index.html         # Sparkasia Studios landing page
solo/index.html     # JeepQuest solo game (localStorage only, no backend yet)
assets/              # Logo and other art — jeepquest-logo.png is the
                       real JcWolF-branded logo (license-plate style,
                       matches PlateQuest's presentation).
LICENSE
README.md
```

## 6. Scoring model

Vehicle categories and points-per-spot live in the `vehicleData` array
near the top of `solo/index.html`'s script. Every tap of "+" logs one
spot and adds that category's point value — this is a running tally,
not a spot-once checklist like PlateQuest's state plates. Current
table:

| Category | Points |
|---|---|
| Jeep — OG (original body style) | 1 |
| Jeep — NOG (Compass, Gladiator, Cherokee, Grand Cherokee, Wagoneer, etc.) | 1 |
| Motorcycle | 3 |
| School bus | 3 |
| Trash/recycling truck | 4 |
| Mail truck | 5 |
| Boat | 6 |
| Law enforcement | 7 |
| Fire truck | 8 |
| Cybertruck | 9 |
| Double mail-truck semi | 10 |
| Batmobile trike (2 front / 1 rear, side-by-side seats) | 20 |
| Tuktuk (3-wheel scooter taxi) | 50 |

If the user changes a point value or adds a category, it's a one-line
edit to `vehicleData` — no other file needs touching for solo mode.

## 7. Player tags are case-sensitive

Same rule as PlateQuest: tags pass through `normalizeTagInputSolo`,
which strips non-alphanumeric characters and caps length at 8 but
**preserves case**. Never `.toUpperCase()` a tag, never
`text-transform: uppercase` a tag input.

## 8. No multiplayer/Firebase yet

Solo mode is deliberately localStorage-only — no Firebase config, no
cloud sync, no admin panel. This was a scope decision (see repo history
of the initial build) to ship something real before taking on
multiplayer. When multiplayer is added, follow PlateQuest's pattern
(own Firebase RTDB namespace, `multiplayer/` directory) rather than
inventing a new architecture.

## 9. Don't add scope

Indie weekend project, same spirit as PlateQuest: no frameworks, no
build step, no package manager. Fix what's asked, don't refactor
nearby code while doing it, don't pre-build abstractions for features
that don't exist yet.

## 10. Mobile-first audience

Same audience as PlateQuest — assume the player is on a phone. Keep
chat responses short, batch tool calls, sanity-check UI at ~375–414px
widths, use `AskUserQuestion` for discrete choices.
