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

Vehicle categories, their points-per-spot, and their tile color live
in the `vehicleData` array near the top of `solo/index.html`'s script.
Each tile in the grid is tapped once per spot (no +/- steppers) and
logs one entry to that category's `pointsLog` — this is a running
tally, not a spot-once checklist like PlateQuest's state plates.
Holding a tile down (≥550ms, `LONG_PRESS_MS`) undoes the single most
recent spot logged for that category, exactly reversing whatever
points it awarded (see bonus rules below) — this is the only undo;
there's no separate minus button. Current point table:

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

If the user changes a point value, tile color, or adds a category,
it's a one-line edit to `vehicleData` — no other file needs touching
for solo mode.

### Bonus rules ("finder bonus")

All three stack independently and are computed fresh on every tap in
`adjustTally()` — nothing is pre-stored as a flag, so undo (popping the
last `pointsLog` entry) always exactly reverses whatever bonuses that
tap earned:

- **First Find (×2, all modes):** the first time *any* spotter logs a
  given category on a trip, that tap scores double points.
- **First Caller (+50%, Family Mode only, 2+ contributors):** the
  first time *this contributor personally* logs a category on the
  trip, they get +50% on top of base points. Requires Family Mode
  with more than one contributor — solo/Challenge trips never trigger
  it since there's only ever one spotter.
- **Streak Bonus (+10 flat, all modes):** every 5th spot of the same
  category (`STREAK_EVERY`/`STREAK_BONUS` constants) on a trip adds a
  flat bonus, independent of the multiplier bonuses above.

A tap's award is `round(basePoints × multiplier) + streakBonus`, where
multiplier starts at 1 and gains +1 (First Find) and/or +0.5 (First
Caller). The score section's subline shows the running bonus total
(`total points − Σ tally×basePoints`) whenever it's nonzero, and every
tap shows a toast naming which bonuses fired.

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

## 11. Update banner — keep `version.json` and both `APP_VERSION`s in sync

The player is on a phone with no keyboard hard-refresh, and GitHub
Pages / mobile browsers cache aggressively. To avoid making players
guess why they're not seeing a new build, both `index.html` and
`solo/index.html` poll `version.json` (on load, every 3 minutes, and
whenever the tab regains visibility) and show a tap-to-refresh banner
when it doesn't match their own hardcoded `APP_VERSION`. Tapping it
reloads with a cache-busting query string, which forces a real fetch.

A version bump is a **three-file change**, same discipline as
PlateQuest's `multiplayer/` versioning:

1. `version.json` — the `"version"` field.
2. `index.html` — the `APP_VERSION` constant near the bottom.
3. `solo/index.html` — the `APP_VERSION` constant near the top of the
   script.

All three must carry the same value (format: `YYYYMMDD<letter>`, see
PlateQuest's CLAUDE.md section 10 for the dating rule — use today's
actual date, don't mechanically bump a letter on a stale one). Missing
one means the banner never fires, or fires and points at a build
that's identical to what's already loaded.

## 12. Mobile-first audience

Same audience as PlateQuest — assume the player is on a phone. Keep
chat responses short, batch tool calls, sanity-check UI at ~375–414px
widths, use `AskUserQuestion` for discrete choices.
