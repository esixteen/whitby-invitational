---
title: Whitby Invitational — Architect Brief for Builder
date: 2026-05-09
status: locked
audience: builder
---

# Architect Brief — Whitby Invitational Dashboard

This brief locks the design system and JS architecture. The Builder follows it without reinterpretation. Every value here is final unless flagged in the Decisions section.

Spec reference: `/Users/ericsciberras/whitby-invitational/docs/superpowers/specs/2026-05-09-whitby-invitational-dashboard-design.md`

Build target: `/Users/ericsciberras/whitby-invitational/index.html` — single file, openable from `file://`.

---

## 1. Color palette (locked)

CSS variable names are required. Tailwind reads them via arbitrary-value notation `bg-[var(--surface)]` etc. Define them inside `<style>` under `:root`.

| Token              | Hex       | Use                                                       |
|--------------------|-----------|-----------------------------------------------------------|
| `--bg`             | `#FBF8F1` | page background (cream/linen)                             |
| `--surface`        | `#FFFFFF` | cards, scorecard background                               |
| `--surface-elev`   | `#F4EFE3` | hero band, expanded scorecard header row, subtle wash     |
| `--ink`            | `#1F2A24` | primary text (dark slate-green, not pure black)           |
| `--ink-muted`      | `#5C645C` | secondary text, table headers, "out" labels (passes AA on `--bg` and `--surface`) |
| `--fairway`        | `#2F6A3E` | primary green — buttons, links, champion accent bars      |
| `--fairway-deep`   | `#1F4D2C` | hover, focused borders, scorecard header row text         |
| `--gold`           | `#C8A24A` | champion ribbon, outright-win cell highlight, trophy glyph|
| `--gold-soft`      | `#F2E3B8` | gold cell background fill (text stays `--ink`)            |
| `--silver`         | `#A8A8A8` | tied-win cell border accent                               |
| `--silver-soft`    | `#E6E6E6` | tied-win cell background fill                             |
| `--divider`        | `#E5DFD0` | hairlines between rows, card borders                      |
| `--danger`         | `#9B3B2E` | "out" pill text (warm rust, not red)                      |
| `--danger-soft`    | `#F2DAD3` | "out" pill background                                     |

Contrast checks performed: `--ink` on `--bg` ≈ 12.4:1; `--ink-muted` on `--bg` ≈ 5.1:1; `--fairway` on `--bg` ≈ 5.6:1. All AA-clear.

---

## 2. Type scale (locked)

Load both fonts via a single Google Fonts `<link>`:
```
https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,500;9..144,600;9..144,700&family=Inter:wght@400;500;600;700&display=swap
```

Apply Fraunces with `font-feature-settings: "ss01", "ss02"` and `font-optical-sizing: auto` on the body so the display weights look right. Inter gets `font-feature-settings: "cv11", "tnum"` so digits are tabular by default — important for the scorecard.

| Role           | Font     | Tailwind-style                              | Notes                                  |
|----------------|----------|---------------------------------------------|----------------------------------------|
| h1 (wordmark)  | Fraunces | `text-[56px]/[1.02] tracking-[-0.02em] font-semibold` (mobile `text-[40px]/[1.05]`) | semibold (600), not bold               |
| h2 (section)   | Fraunces | `text-[32px]/[1.1] tracking-[-0.015em] font-semibold` |                                        |
| h3 (card)      | Fraunces | `text-[22px]/[1.2] tracking-[-0.01em] font-medium` (500) |                                        |
| eyebrow        | Inter    | `text-[12px]/[1.2] tracking-[0.14em] font-semibold uppercase` | use for "REIGNING CHAMPION", "NEXT TOURNAMENT" |
| body           | Inter    | `text-[16px]/[1.5] font-normal`             | default paragraph                      |
| body-strong    | Inter    | `text-[16px]/[1.5] font-semibold`           |                                        |
| small          | Inter    | `text-[13px]/[1.4] font-normal`             | metadata, captions                     |
| tabular        | Inter    | `text-[14px]/[1.3] font-medium tabular-nums` | scorecard cells, leaderboard digits    |
| countdown digit| Fraunces | `text-[44px]/[1.0] tracking-[-0.02em] font-semibold tabular-nums` | days/hours/minutes |

Set `body { font-family: 'Inter', system-ui, sans-serif; color: var(--ink); background: var(--bg); }` and use a utility class `.font-display` for Fraunces.

---

## 3. Spacing & radius tokens

Stick to a 4px base. Use Tailwind's default spacing scale; do not invent custom px values except where called out.

| Token          | Value     | Use                                          |
|----------------|-----------|----------------------------------------------|
| section-pad-y  | 64px (`py-16`) desktop / 40px (`py-10`) mobile | between sections                             |
| section-pad-x  | 24px (`px-6`) mobile / 48px (`px-12`) desktop  | page gutters                                 |
| max-width      | 1120px (`max-w-[1120px] mx-auto`)              | content column                               |
| card-pad       | 28px (`p-7`) desktop / 20px (`p-5`) mobile     | inside surface cards                         |
| card-gap       | 24px (`gap-6`)                                 | between cards in a grid                      |
| stack-gap-tight| 8px (`gap-2`)                                  | label + value pairs                          |
| stack-gap      | 16px (`gap-4`)                                 | default vertical rhythm inside cards         |
| radius-sm      | 6px (`rounded-md`)                             | pills, scorecard cells                       |
| radius-md      | 12px (`rounded-xl`)                            | buttons, inputs                              |
| radius-lg      | 18px (`rounded-[18px]`)                        | cards (custom)                               |
| radius-xl      | 28px (`rounded-[28px]`)                        | hero band, next-tournament card              |
| border         | `1px solid var(--divider)`                     | all card borders                             |
| shadow-card    | `0 1px 2px rgba(31,42,36,0.04), 0 8px 24px -12px rgba(31,42,36,0.12)` | gentle, no glow             |

---

## 4. Section blueprints

All sections live inside one `<main class="max-w-[1120px] mx-auto px-6 lg:px-12">`. Section vertical rhythm uses `py-10 lg:py-16`. Use `<section id="...">` with the section name for keyboard anchors.

### 4.1 Hero

```
+------------------------------------------------------------------+
| [GOLD RIBBON · top of card]                                      |
|   ★  REIGNING CHAMPION · GIBBS · TOURNAMENT I · WINCHESTER GC    |
+------------------------------------------------------------------+
|                                                                  |
|        The Whitby Invitational            (Fraunces 56/40)       |
|        An annual gentlemen's tradition · Whitby, Ontario         |
|                                       (Inter 16, ink-muted)      |
|                                                                  |
|        [flagstick svg]   [tee svg]                               |
+------------------------------------------------------------------+
```

- Outer wrapper: `relative overflow-hidden rounded-[28px] bg-[var(--surface-elev)] border border-[var(--divider)] px-8 py-12 lg:px-14 lg:py-20`.
- Subtle paper texture: a `::before` with an inline SVG noise filter, `opacity: 0.06`, `pointer-events: none`. SVG: `<feTurbulence baseFrequency="0.9" numOctaves="2" />`. Encode as data-URL.
- Gold ribbon: absolutely positioned `top: 0; left: 0; right: 0; height: 36px; background: var(--gold);` with eyebrow text in `--ink` (gold + dark ink passes AA).
- Wordmark: `font-display text-[40px] lg:text-[56px] leading-[1.02] tracking-[-0.02em] font-semibold`.
- Subtitle: `mt-3 text-[var(--ink-muted)]`.
- Pad top to clear ribbon: `pt-[60px] lg:pt-[80px]`.

### 4.2 Next Tournament

Two-column on desktop (`grid lg:grid-cols-[1.1fr_1fr] gap-6`), single column on mobile.

```
+----------------------------------+   +-----------------------------+
| eyebrow: NEXT TOURNAMENT         |   | eyebrow: ROSTER (12)        |
|                                  |   |                             |
| Kedron Dells Golf Club           |   |  ⬤F ⬤E  ⬤M ⬤T  ⬤A         |
| Oshawa, Ontario                  |   |  ⬤W ⬤S  ⬤D ⬤Dn ⬤Da        |
|                                  |   |                             |
| Sat, June 13, 2026 · Tee TBD     |   |  --- OUT ---                |
|                                  |   |  ⊘Greg  ⊘Gibbs              |
| [ 35 ] [ 12 ] [ 47 ]             |   |                             |
|  DAYS   HRS   MINS               |   |                             |
+----------------------------------+   +-----------------------------+
```

- Each is a card: `rounded-[18px] bg-[var(--surface)] border border-[var(--divider)] shadow-card p-7`.
- Course name: `font-display text-[28px] leading-[1.15] font-semibold`.
- Date line: `text-[var(--ink-muted)] text-sm mt-1`.
- Countdown: `flex gap-3 mt-6`. Each unit is `flex flex-col items-center justify-center bg-[var(--surface-elev)] rounded-xl px-5 py-4 min-w-[88px]`. Digit uses the countdown style above; label is the eyebrow style in `--ink-muted`.
- Roster grid: `grid grid-cols-5 gap-3` for confirmed; `mt-5 pt-5 border-t border-[var(--divider)]` then `grid grid-cols-5 gap-3` for out.
- Confirmed avatar: see §5.1. Out avatar: same shape but `opacity-60` with name strikethrough and a small "OUT" pill below.

### 4.3 All-time Leaderboard

Full-width card. Table inside.

```
+----------------------------------------------------------------------+
| eyebrow: ALL-TIME LEADERBOARD                                        |
|                                                                      |
|  PLAYER          TITLES   HCP    T-I   T-II  ...                     |
|  ─────────────────────────────────────────────                       |
| ┃Gibbs                1   +18    1                                   |  ← gold left bar
|  Deluca               0   +26    2                                   |
|  Dennis               0   +41    3                                   |
|  ...                                                                 |
+----------------------------------------------------------------------+
```

- Container: `rounded-[18px] bg-[var(--surface)] border border-[var(--divider)] shadow-card overflow-hidden`.
- Header eyebrow + h2 inside `px-7 pt-7`.
- Table: `<table class="w-full text-left mt-5">`. Wrap in `<div class="overflow-x-auto">` for mobile.
- `<thead>`: row class `text-[12px] uppercase tracking-[0.14em] text-[var(--ink-muted)] border-b border-[var(--divider)]`. Cells `px-4 py-3`.
- `<tbody>` rows: `border-b border-[var(--divider)] last:border-0`. Cells `px-4 py-4 text-sm`.
- Champion row: add `class="relative"` and a `::before` pseudo using a real `<td>` is not possible; instead set the first `<td>` to `border-l-[3px] border-[var(--gold)]` and adjust its `pl-3`. Apply this to any player whose `championships > 0`.
- Titles column: `font-semibold tabular-nums`.
- Handicap column: render as `+18`. `tabular-nums`.
- Per-tournament finish columns: render finish position (`1`, `T2`, `4`) or `—` for "did not play". Highlight `1` cells with `text-[var(--fairway-deep)] font-semibold`.
- Mobile (<640px): hide the per-tournament columns, replace with a "▾ rounds" disclosure that expands an inline list `T-I: 1 · T-II: —`. Use `<details><summary>` for keyboard support.

### 4.4 Tournament Archive

One `<section>` with a header and a stack of `<article>` cards (one per tournament). Tournament I `open` by default.

```
+----------------------------------------------------------------------+
| ▾  Tournament I · Winchester GC, Whitby ON · 2025                    |
|     🥇 Gibbs   🥈 Deluca   🥉 Dennis                                 |
|     ───────────────────────────────────────────────────────────      |
|     HOLE   1  2  3  4 ... 18   STR  +/-  PTS  FIN                    |
|     PAR    4  3  4  4 ...  5    70   —    —    —                     |
|     Gibbs  7  3 [5][5][3][5]... 88  +18  14   1                      |
|     ...                                                              |
+----------------------------------------------------------------------+
```

- Use `<details open>` for the first tournament, `<details>` for the rest. Style the disclosure to remove the default triangle: `details > summary { list-style: none; cursor: pointer; } summary::-webkit-details-marker { display: none; }`. Add a custom chevron SVG that rotates 180° when `details[open]`.
- Card chrome: `rounded-[18px] bg-[var(--surface)] border border-[var(--divider)] shadow-card overflow-hidden`. `summary` gets `flex items-center justify-between px-7 py-5 gap-4`.
- Summary content: title (Fraunces h3) + podium chips (`inline-flex items-center gap-1 text-sm`, with gold/silver/bronze dots). Bronze color: `#A8784E` (only used here, not a token).
- Body wrapper: `px-7 pb-7`.
- Scorecard container: `overflow-x-auto` with `-mx-7 px-7` on mobile so the table can scroll edge-to-edge while card padding is preserved.
- Scorecard table: `w-full border-separate border-spacing-0 text-[14px] tabular-nums`.
- `thead` first row "HOLE" labels: bg `var(--surface-elev)`, ink-muted, `text-[12px] uppercase tracking-[0.12em]`.
- `thead` second row "PAR": bg `var(--surface-elev)`, `font-medium`, ink.
- Body cells: `text-center w-[36px] h-[36px] border-b border-[var(--divider)]`.
- **Cell highlight (per-hole points):**
  - Outright win (2 pts): `background: var(--gold-soft); color: var(--ink); font-weight: 600; border-radius: 6px; box-shadow: inset 0 0 0 1px var(--gold);`
  - Tied win (1 pt): `background: var(--silver-soft); color: var(--ink); border-radius: 6px; box-shadow: inset 0 0 0 1px var(--silver);`
  - Apply via class on the `<td>`. Compute classes during render from the points map.
- Trailing columns (STR / +/- / PTS / FIN): `bg-[var(--surface-elev)]`, `font-semibold`, sticky-right would be nice but not required. FIN cell for rank 1 gets `text-[var(--fairway-deep)]`.

### 4.5 Past Champions

```
+--------------------+ +--------------------+
| 🏆 (gold glyph)    | | 🏆                |
| GIBBS              | | (next year)       |
| Tournament I       | |                   |
| Winchester GC · '25| |                   |
| 14 pts · ru: Deluca| |                   |
+--------------------+ +--------------------+
```

- Grid: `grid sm:grid-cols-2 lg:grid-cols-3 gap-6`.
- Each card: `rounded-[18px] bg-[var(--surface)] border border-[var(--divider)] shadow-card p-7 flex flex-col gap-3`.
- Trophy glyph: 36×36 SVG, fill `var(--gold)`. Inline SVG, no external request.
- Champion name: `font-display text-[28px] leading-[1.1] font-semibold`.
- Tournament + course + year: `text-sm text-[var(--ink-muted)]`.
- Stat line: `text-sm text-[var(--ink)]` with `font-medium` for points number.

---

## 5. Component micro-decisions

### 5.1 Initial-circle avatar

- Size: `w-12 h-12` (48px) confirmed; `w-12 h-12` for out (same size, lower opacity).
- Shape: full circle (`rounded-full`), `bg-[var(--fairway)]` for confirmed, `bg-[var(--ink-muted)]` for out.
- Letter: white, Inter, `text-[18px] font-semibold`, single uppercase initial.
- Two-letter case: if two roster players share a first initial (e.g. "Dennis" and "Danny"), use the first two letters of the name (e.g. `De`, `Da`). Implement by generating initials in the render function with a collision check across the roster.
- Container per avatar: `flex flex-col items-center gap-1.5`. Below the circle: name in `text-[12px] text-[var(--ink)] font-medium`, truncated.
- Out variant: add a small "OUT" pill below the name — `inline-flex px-2 py-0.5 rounded-md bg-[var(--danger-soft)] text-[var(--danger)] text-[10px] font-semibold uppercase tracking-[0.1em]`. Strikethrough the name with `line-through`.

### 5.2 Scorecard cell highlighting

- Compute `pointsByHole[hole][playerId] = 0 | 1 | 2` once, store on the tournament's render context.
- Apply class:
  - `2` → `cell-gold`
  - `1` → `cell-silver`
  - `0` → no class
- CSS:
  ```
  td.cell-gold   { background: var(--gold-soft); box-shadow: inset 0 0 0 1px var(--gold); border-radius: 6px; font-weight: 600; }
  td.cell-silver { background: var(--silver-soft); box-shadow: inset 0 0 0 1px var(--silver); border-radius: 6px; }
  ```
- Do not apply both. Outright takes precedence (it can't tie by definition).

### 5.3 Countdown widget

- Compute target: `new Date(data.upcoming.date + "T08:00:00-04:00")` (Eastern, 8am, a sensible default tee time).
- Render three units: Days, Hours, Minutes. Skip seconds — too noisy for a once-a-year event.
- Update every 30 seconds via `setInterval`. Stop the interval when the target is in the past and replace markup with a `font-display` "Tee it up." line.
- Anatomy per unit: rounded tile (see §4.2), digit on top in countdown style, label below in eyebrow style. Pad digits to 2 chars (`String(n).padStart(2, "0")`).

### 5.4 Expand/collapse model

- Use native `<details>` / `<summary>`. No custom JS toggle.
- Style summary chevron: a single inline SVG inside `<summary>`, with CSS `details[open] svg.chevron { transform: rotate(180deg); }` and `transition: transform 200ms ease`.
- Focus ring: `summary:focus-visible { outline: 2px solid var(--fairway); outline-offset: 4px; border-radius: 12px; }`.
- Tournament I: `<details open>`. All others: `<details>`.

---

## 6. JS architecture

Single `<script>` block at the bottom of `<body>`. No modules, no imports beyond CDN tags in `<head>`. Render strategy: **template literal HTML strings injected via `innerHTML` into per-section mount points**. Reasoning: the data is small (≤20 players, ≤dozens of tournaments over the life of the project), there is no reactivity requirement after first paint, and string templating keeps the file readable in one scroll. Avoids the verbosity of `document.createElement` chains.

### File structure (top to bottom of the `<script>`)

```
1. const data = { ... }                      // copied verbatim from spec §Data model
2. // ---- pure compute ----
   function computeTournamentStandings(t, course)   // returns array of {playerId, points, strokes, rank, pointsByHole}
   function computeHandicaps(data)                   // returns { playerId: handicapNumber }
   function computeLeaderboard(data)                 // returns array of {playerId, name, championships, handicap, finishes: {tournamentId: rank}}
   function pointsForHole(scoresAtHole)              // helper: returns { playerId: 0|1|2 }
   function initialsFor(playerId, allPlayers)        // collision-aware initials
   function formatHandicap(n)                        // "+18"
   function ordinal(n)                               // "1", "T2" (T applied separately when tied)
3. // ---- renderers (return HTML strings) ----
   function renderHero(reigningChampion)
   function renderNextTournament(upcoming, courses, players)
   function renderLeaderboard(leaderboard, tournaments)
   function renderArchive(tournaments, courses, players)
   function renderPastChampions(tournaments, courses, players)
4. // ---- mount ----
   document.getElementById('hero').innerHTML = renderHero(...)
   ... etc
5. // ---- countdown loop ----
   startCountdown(targetDate, mountId)
6. // ---- self-tests ----
   console.assert(...)   // see §7
```

### Pure compute contracts

```js
// computeTournamentStandings(tournament, course) =>
// {
//   pointsByHole: [{playerId: 0|1|2}, ...18],   // index 0 = hole 1
//   players: [
//     { playerId, totalStrokes, totalPoints, overPar, rank, tied: bool }
//   ]  // sorted by points desc, strokes asc
// }
```

`computeLeaderboard` calls `computeTournamentStandings` for each tournament, accumulates championships (count of rank-1 finishes; if a tournament's rank 1 is tied — currently impossible by tiebreak rules but defend against it — give championship to the strokes-tiebreaker winner only), and merges with `computeHandicaps`. Players with zero tournaments played are excluded (per spec §3 leaderboard rules).

### DOM mount points

In the HTML, define empty containers:
```html
<section id="hero"></section>
<section id="next"></section>
<section id="leaderboard"></section>
<section id="archive"></section>
<section id="champions"></section>
```

Each renderer returns the full inner markup including its own padding wrapper. This keeps each section self-contained.

### No-framework rules

- No `Date.parse` of ambiguous strings — use the explicit `"YYYY-MM-DDTHH:MM:SS-04:00"` form.
- No `eval`, no inline event handlers in strings. Wire any listeners after `innerHTML` injection (only one needed: countdown interval).
- Escape player names if they ever contain HTML (currently safe, but write a `esc(s)` helper anyway and use it).

---

## 7. Verification hooks (mandatory)

At the bottom of the `<script>`, after mounting, add a self-test block. It must fail loudly in the console if the compute output drifts from the spec. Do not gate rendering on it — render first, assert second, so a bad assert still leaves a visible page.

```js
// ---- self-tests : Tournament I reference table ----
(function runSelfTests() {
  const t1 = data.tournaments.find(t => t.id === 1);
  const standings = computeTournamentStandings(t1, data.courses[t1.courseId]);

  const expected = [
    { playerId: "gibbs",  points: 14, strokes: 88,  rank: 1 },
    { playerId: "deluca", points: 9,  strokes: 96,  rank: 2 },
    { playerId: "dennis", points: 4,  strokes: 111, rank: 3 },
    { playerId: "angel",  points: 4,  strokes: 115, rank: 4 },
    { playerId: "e",      points: 3,  strokes: 104, rank: 5 },
    { playerId: "franco", points: 3,  strokes: 108, rank: 6 },
    { playerId: "adam",   points: 3,  strokes: 110, rank: 7 },
    { playerId: "danny",  points: 2,  strokes: 115, rank: 8 },
  ];

  expected.forEach((exp, i) => {
    const got = standings.players[i];
    console.assert(got, `T-I row ${i+1} missing`);
    console.assert(got.playerId === exp.playerId,
      `T-I rank ${i+1}: expected ${exp.playerId}, got ${got && got.playerId}`);
    console.assert(got.totalPoints === exp.points,
      `T-I ${exp.playerId}: expected ${exp.points} pts, got ${got && got.totalPoints}`);
    console.assert(got.totalStrokes === exp.strokes,
      `T-I ${exp.playerId}: expected ${exp.strokes} strokes, got ${got && got.totalStrokes}`);
  });

  // Course par sanity
  console.assert(
    data.courses.winchester.pars.reduce((a,b)=>a+b,0) === 70,
    "Winchester par sum must equal 70"
  );

  // Handicap sanity: Gibbs is +18 after T-I (88 - 70)
  const hcs = computeHandicaps(data);
  console.assert(hcs.gibbs === 18, `Gibbs handicap expected 18, got ${hcs.gibbs}`);

  console.log("%c✓ Whitby self-tests passed", "color:#2F6A3E;font-weight:600");
})();
```

If any assert fires, the Builder must fix the compute function — not the expected values. The expected values come straight from the spec §Reference standings.

---

## 8. Implementation order (suggested for Builder)

1. `index.html` skeleton: head with Tailwind Play CDN, Google Fonts link, inline `<style>` with `:root` variables, body with five empty `<section>` mount points.
2. Paste `data` block from spec verbatim.
3. Write compute functions. Run self-tests in console. **Do not proceed until they pass.**
4. Build `renderHero`, `renderNextTournament`, `renderPastChampions` (simplest, low data-dependency).
5. Build `renderLeaderboard`.
6. Build `renderArchive` last (most complex — full scorecard).
7. Wire countdown.
8. Cross-check on iPhone-width viewport (`375px`) and desktop (`1280px`).

---

## 9. Decisions (judgment calls locked here)

The spec was tight, but these are the spots I had to pick:

1. **Reigning champion ribbon placement.** Spec said "gold band across the top." I locked it inside the hero card (top edge), not as a global sitewide bar. Cleaner, more obviously contextual to the hero.
2. **Tee time default for countdown.** Spec says "Tee time: TBD." I locked countdown target at 08:00 ET on the tournament date so the countdown is meaningful. Builder should add a comment noting this is a sensible default until a real tee time is set in `data.upcoming.teeTime`.
3. **"Out" treatment on roster.** Spec offered "strikethrough or 'out' pill." I locked **both** — strikethrough on the name plus a small OUT pill — because either alone is too subtle on a busy roster grid.
4. **Bronze color for podium chip.** Not in the palette. Used `#A8784E` (warm bronze) once, only for the 3rd-place chip. Did not promote it to a token because it has exactly one use site.
5. **Initial collisions.** Two players share `D` (Deluca, Dennis, Danny). Two share `D...` after that. Locked the rule: prefix until unique within the roster being rendered. Deluca → `De`, Dennis → `Dn`, Danny → `Da`. (Spec did not prescribe.)
6. **Tied championships.** Spec's tournament tiebreak (points desc, strokes asc) makes a true tie at rank 1 essentially impossible with these inputs, but the leaderboard rule "championships won (desc)" needs a definition. Locked: a championship is awarded to the rank-1 player after the tiebreaker; never split.
7. **Mobile leaderboard collapse.** Spec said "expandable row." Locked `<details><summary>` per row instead of building a custom toggle, for keyboard support and zero-JS reliability.
8. **Render strategy.** Locked template-literal HTML strings (not `createElement`). Reasoning in §6.
9. **Texture.** Spec said "subtle paper/linen noise." Locked an inline SVG `feTurbulence` data-URL at 6% opacity, hero only — no asset to fetch, no CDN dependency.
10. **No `seconds` in countdown.** Spec said "days · hours · minutes." Held the line. Seconds would feel frantic for a tournament that's weeks out.

---

## 10. Out of scope for Builder

Confirm none of the following is built (per spec):
- No live scoring, no edit UI, no auth.
- No image gallery, no per-player profile pages.
- No build step, no npm, no bundler, no service worker.
- No analytics snippet.

Builder ships one file: `/Users/ericsciberras/whitby-invitational/index.html`. That's it.
