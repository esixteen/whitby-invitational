---
title: Whitby Invitational Dashboard — Redesign Brief
date: 2026-05-09
status: in_flight
direction: The Modern Major
---

# Redesign brief — "The Modern Major"

The first dashboard ships and works. This redesign pushes the visual quality from "tidy" to "wicked" — the user wants to impress his buddies. Direction is **The Modern Major** from `docs/superpowers/specs/2026-05-09-design-research.md`.

## Mood, in one sentence

**The morning-after Athletic article about your friend group's tournament** — editorial sports magazine layout, big confident serif type, generous whitespace, fairway-green hero blocks with gold stat highlights. Polished, current, screen-grabs cleanly into the group chat.

## Authoritative inputs (read these first)

1. `docs/superpowers/specs/2026-05-09-whitby-invitational-dashboard-design.md` — original spec (data model, compute rules, reference standings).
2. `docs/superpowers/plans/2026-05-09-architect-brief.md` — locked palette, typography, JS architecture.
3. `docs/superpowers/specs/2026-05-09-design-research.md` — cultural research, three direction concepts, recommendation.
4. `index.html` — current build. Compute logic and self-test asserts must continue to pass after the redesign.

## Non-negotiables

- **All existing compute logic and asserts must still pass.** Champion = Gibbs, points = 14, strokes = 88, etc. Don't refactor compute functions.
- **Light theme.** Cream + fairway green + gold + soft slate. Don't drift to dark or to a different palette family.
- **Single self-contained `index.html`.** Tailwind via Play CDN, Google Fonts via `<link>`, all images via absolute URLs (Unsplash) or local paths under `~/whitby-invitational/img/`. No build step, no npm.
- **Mobile-first responsive.** Layouts must hold at 375px, 768px, 1280px+.
- **Use the `frontend-design:frontend-design` skill** when you start. The whole point of this redesign is design quality.

## What's changing

### 1. Hero — new treatment
Editorial cover. Full-bleed cream background with subtle paper grain. Wordmark stays Fraunces but goes BIG (clamp(56px, 9vw, 132px)) with `EST. 2025` lockup on the right side, set in small Inter caps with a thin gold rule. Below the wordmark: a horizontal **stat strip** in giant numbers — `1` Tournaments | `8` Players | `1` Champion | `35` Days to next tee. Numbers in Fraunces 64px+, labels in Inter caps. The reigning-champion ribbon stays but becomes a centered gold pill ("Reigning Champion · Gibbs") with more breathing room.

### 2. Champion Spotlight — new section, replaces inline ribbon
Full-width card directly under the hero. Left side: oversized champion name in Fraunces 7–9rem with thin gold underline. Right side: a 60/40 photo (reserve a `<picture>` with a stock golf-celebration shot for now, ~/whitby-invitational/img/group-photo.jpg as the actual file path the user will drop). Stats below the name: `14 pts · 88 strokes · +18 over par · Winchester GC, Sept 1 2025`. Quote line under the photo: italic Fraunces, room for the user to fill in the champion's victory quote (default text: *"You guys are gonna have to do better next time."* — placeholder, user-editable).

### 3. The Hardware Cabinet — new section (Awards)
Four award cards in a responsive grid. Light, cream cards with thin gold borders, generous padding, big serif numbers, small Inter caps labels.
- **The Champion** — Gibbs · 14 pts · gold trophy SVG.
- **Runner-Up** — Deluca · 9 pts · silver trophy SVG.
- **The Sweater of Shame** — Danny · 2 pts · woolly sweater SVG icon. Subtitle: *"Must wear to the next year's first tee."*
- **The Blow-Up Hole** — compute the worst single-hole score in the tournament; if multiple players tied at the max, show both names and the hole(s); icon: a fairway-green explosion glyph. Subtitle: *"Take a moment of silence."*

Each card has a tiny bottom row showing the player's initials avatar + their handicap.

**Compute logic to add** (one new pure function):
```js
function computeBlowUp(tournament) {
  // returns { score, holes: [{player, holeIndex}], displayLine: "Danny — 10 on hole 16" or "Dennis & Danny — 10s" }
}
```
Add a `console.assert` for Tournament I that the worst single-hole is `10` and includes Dennis and Danny.

### 4. Moments Feed — new section
Horizontal scrollable feed of captioned cards. Each card: photo (or generated SVG poster if no photo), one-line caption in Fraunces, sub-line in Inter (round number, hole). User-editable JS data block:

```js
moments: [
  { tournamentId: 1, hole: 17, caption: "Gibbs cans a 25-footer for the lead", img: "img/moments/m1.jpg" },
  // user fills in more
]
```

For the initial build, populate 4 placeholder moments with stock Unsplash imagery. Make it dead obvious in the data block where the user adds their own.

### 5. Next Tournament — refined, not replaced
Keep the existing Kedron Dells card and countdown logic. Visual upgrade: course header photo at top (full-bleed, ~280px tall, subtle gold gradient overlay at bottom for text legibility), course name in Fraunces over the photo, countdown chips become bigger and more confident. Roster grid keeps the avatar circles but sizes them up to 56px and groups confirmed/out into two clear rows with a thin gold rule between.

### 6. Leaderboard — refined
Same compute. Visual changes only: bigger row gutters, bigger rank numbers (Fraunces 32px), gold trophy glyph for #1, silver for #2, tarnished bronze for #3. Hover state lifts the row by 1px with a soft shadow. Keep the per-tournament finish columns (T-1, T-2…) but change column header style to small Inter caps.

### 7. Tournament Archive — refined
Course header photo at the top of each card (same treatment as Next Tournament). Add a small **Round Story** subsection above the scorecard: 3 auto-generated bullet stat lines like *"Lead changed 2 times. Gibbs took the lead at hole 7 and never looked back. 4 players carded a 10."* The scorecard table itself stays — gold/silver hole highlights — but pad the cells more, lighten the borders, and add a subtle alternating-row background.

### 8. Past Champions — refined
Trophy gallery becomes a **Champions Wall**. One card per past champion (just Gibbs for now). Big serif year + champion name on top, course + score below, room for a champion quote. Placeholder for a champion portrait (square aspect ratio, top-aligned). When a future tournament is added, this wall accumulates.

### Hero stat numbers — compute
The hero stat strip values come from data:
- Tournaments played = `data.tournaments.length`
- Players ever-played = unique players across all tournament `scores` keys
- Champions = unique winners across all tournaments
- Days to next tee = same calc as countdown

## Image strategy

**The user will drop one group photo into the conversation.** Save it to `~/whitby-invitational/img/group-photo.jpg`. Use it in the Champion Spotlight (right-side photo) for now.

**Stock imagery (Unsplash, royalty-free).** Source 5–7 photos and download them locally to `~/whitby-invitational/img/stock/`:
- 1× hero background option (fairway in golden hour) — kept subtle/optional.
- 1× Winchester GC card photo (substitute: any green Ontario fairway).
- 1× Kedron Dells card photo (same, generic Ontario fairway).
- 4× "moments" feed placeholders (a chip shot, a beer/clubhouse shot, a putt rolling toward the hole, group-of-friends-walking shot).

Use the Unsplash search API or `WebSearch` for "Unsplash golf fairway Ontario", "golf chip shot close up", etc. Download via curl. Cite source URL in a small `IMAGE_CREDITS.md` file in the project root.

Image dimensions: don't ship 4000×3000 originals. Use ImageMagick (`sips` on macOS) to resize: hero ≤ 1920w, archive cards ≤ 1600w, moments ≤ 800w. JPEG 80%.

If the user's group photo isn't dropped before you finish, leave a `<picture>` placeholder with a clear DOM comment `<!-- USER PHOTO GOES HERE: img/group-photo.jpg -->` and use a stock placeholder.

## Motion

Subtle. Section fade-up on scroll (IntersectionObserver, 200ms ease-out, once only). Champion name does a single 400ms scale-in on load. Moments feed scroll snaps to cards on touch. No bounce, no parallax, no scroll-jacking.

## What stays exactly the same

- The `data` object structure and contents (only ADD `moments: []` and any optional copy fields like `championQuote`).
- All three compute functions (`computeTournamentStandings`, `computeHandicaps`, `computeLeaderboard`).
- The self-test assert block — extend it for the new `computeBlowUp` function.
- The locked color tokens from the Architect brief (extend, don't replace).
- The `<details>`/`<summary>` pattern for archive expand/collapse.

## Verification

Before reporting done:
1. `console.assert` block still passes (extend with blow-up assertion).
2. Open the file in a browser via `open`. Visit each section. Verify:
   - All 8 sections render: Hero, Champion Spotlight, Hardware Cabinet (awards), Moments Feed, Next Tournament, Leaderboard, Tournament Archive, Champions Wall.
   - No red error banner.
   - Photos load (or placeholders show with the comment hint).
   - Countdown shows ~35 days.
   - Mobile (375px) layout: nothing horizontally overflows except the moments feed (intentional).
3. Image files exist on disk in `~/whitby-invitational/img/`. Total image weight < 4MB combined.
4. `IMAGE_CREDITS.md` exists with a row per stock image and its Unsplash source URL.

## Done definition

A single self-contained `index.html` plus a small `img/` directory and `IMAGE_CREDITS.md`. Looks like a magazine cover on desktop, holds together cleanly on mobile, and the user can swap in their own group photo by replacing one file.
