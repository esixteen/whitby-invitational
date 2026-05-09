---
title: Whitby Invitational Dashboard — Design
date: 2026-05-09
status: approved
---

# The Whitby Invitational — Dashboard Design

A static, single-file dashboard for an annual golf tournament hosted in Whitby, Ontario, Canada. Light theme. Fun. Easy to update year over year.

## Goals

- Celebrate the tournament with a clubhouse-meets-modern visual identity.
- Surface the four things that matter: reigning champion, next event, all-time leaderboard, and a per-tournament archive with full scorecards.
- Keep updates trivial: drop a new tournament's raw scores into a JS array; the dashboard recomputes points, standings, and handicaps automatically.

## Non-goals

- No backend, no auth, no live scoring (yet).
- No build tooling. No npm. No bundler.
- No mobile app or PWA install flow. Just a webpage that looks great on phone and desktop.

## Stack

- Single `index.html` with embedded CSS + vanilla JS.
- Tailwind via CDN (Play CDN) for utility classes.
- Google Fonts: **Fraunces** (display serif) + **Inter** (body sans).
- Zero dependencies otherwise. Open the file in a browser to view; deployable to any static host.

## Project layout

```
~/whitby-invitational/
├── index.html              # the whole dashboard
├── docs/superpowers/specs/
│   └── 2026-05-09-whitby-invitational-dashboard-design.md
└── .gitignore
```

## Visual direction

- **Palette:** cream / linen background (`#FBF8F1`), fairway green primary (`#2F6A3E`), deeper bottle-green accent (`#1F4D2C`), warm gold for champion highlights (`#C8A24A`), muted slate text (`#2A2F2A`).
- **Texture:** very subtle paper/linen noise behind the hero, otherwise flat.
- **Type:** Fraunces for headings (sub-tight tracking, slight optical-size weight), Inter for body and tabular data.
- **Iconography:** small SVG glyphs — flagstick, golf ball, trophy, tee. Used sparingly as accents, never as decoration.
- **Motion:** subtle. Counter ticks for the countdown, a soft fade-in for sections on load. No bounce. No emojis as load-bearing UI.

## Sections (top → bottom)

### 1. Hero
- Wordmark: "The Whitby Invitational" in Fraunces.
- Subtitle: "An annual gentlemen's tradition · Whitby, Ontario".
- Reigning Champion ribbon: gold band across the top — *"Reigning Champion: Gibbs · Tournament I, Winchester GC"*.

### 2. Next Tournament card
- Course: **Kedron Dells Golf Club**, Oshawa, Ontario.
- Date: **June 13, 2026**. Tee time: TBD.
- Live countdown (days · hours · minutes) computed in-browser.
- Roster grid: 10 confirmed players (Franco, E, Morsillo, Traikos, Angel, Wells, Sean, Deluca, Dennis, Danny) with initial-circle avatars; 2 out (Greg, Gibbs) shown muted with a strikethrough or "out" pill.

### 3. All-time Leaderboard
- Primary sort: **championships won** (desc).
- Tiebreaker: **handicap** (asc, lower is better).
- Columns: Player · Titles · Handicap · per-tournament finish (T1, T2, …).
- Champion row gets a gold accent bar on the left edge.
- Players who have never played a tournament are not shown (next-tournament-only players are listed in the roster card, not the leaderboard, until they actually play).

### 4. Tournament Archive
- One expandable card per completed tournament. Tournament I open by default.
- Card header: "Tournament I · Winchester Golf Club, Whitby ON · 2025" + podium chip (1st/2nd/3rd).
- Full scorecard: 8 players × 18 holes, with par row at top.
- Per-hole highlight: gold cell = outright winner (2 pts), silver cell = tied winner (1 pt).
- Trailing columns: total strokes, over par, points, finish.

### 5. Past Champions
- Trophy gallery. One card per past champion. Shows: trophy glyph, champion name, tournament name, course, year, points, runner-up.

## Data model

```js
const data = {
  players: {
    deluca:   { name: "Deluca",   active: true },
    franco:   { name: "Franco",   active: true },
    dennis:   { name: "Dennis",   active: true },
    danny:    { name: "Danny",    active: true },
    gibbs:    { name: "Gibbs",    active: true },
    angel:    { name: "Angel",    active: true },
    e:        { name: "E",        active: true },
    adam:     { name: "Adam",     active: true },
    morsillo: { name: "Morsillo", active: true },
    traikos:  { name: "Traikos",  active: true },
    wells:    { name: "Wells",    active: true },
    sean:     { name: "Sean",     active: true },
    greg:     { name: "Greg",     active: true },
  },

  courses: {
    winchester: {
      name: "Winchester Golf Club",
      location: "Whitby, ON",
      pars: [4,3,4,4,3,4,5,4,3,4,4,4,4,4,4,4,3,5], // total 70
    },
    kedron: {
      name: "Kedron Dells Golf Club",
      location: "Oshawa, ON",
      // pars unknown until played
    },
  },

  tournaments: [
    {
      id: 1,
      roman: "I",
      year: 2025,
      courseId: "winchester",
      date: "2025-08-XX", // exact date TBD; not displayed
      scores: {
        deluca: [5,5,4,4,2,6,7,7,3,5,5,4,6,7,7,7,4,8],
        franco: [7,6,6,5,5,5,6,6,5,5,6,4,7,9,7,7,4,8],
        dennis: [4,5,6,6,4,7,8,6,5,3,4,5,7,8,8,10,5,10],
        danny:  [5,5,7,6,5,6,10,6,5,5,4,4,6,6,7,10,8,10],
        gibbs:  [7,3,5,5,3,5,5,3,5,3,8,5,4,6,6,7,3,5],
        angel:  [7,7,6,6,6,6,6,6,6,5,4,9,8,5,6,7,8,7],
        e:      [6,5,6,4,7,6,7,8,4,4,6,6,5,7,5,8,4,6],
        adam:   [7,5,7,4,5,5,9,5,5,5,6,8,4,7,7,9,5,7],
      },
    },
  ],

  upcoming: {
    courseId: "kedron",
    date: "2026-06-13",
    teeTime: null,
    confirmed: ["franco","e","morsillo","traikos","angel","wells","sean","deluca","dennis","danny"],
    out: ["greg","gibbs"],
  },
};
```

## Computation rules

### Per-hole points
For each hole, the lowest gross score wins.
- If exactly one player has the lowest score → that player gets **2 points** ("outright").
- If multiple players tie for the lowest → each tied player gets **1 point**.
- All other players get 0.

### Tournament standings
1. Sort players by total points (desc).
2. Ties broken by total strokes (asc).
3. The #1 player is the **champion** for that tournament.

### Handicap
For each tournament a player has played, compute `over_par = total_strokes − course_par`. Handicap = average of all `over_par` values across played tournaments. Lower is better. Displayed as `+N`.

### Leaderboard sort
1. Championships won (desc).
2. Handicap (asc).
3. Player name (alpha) as a stable last-resort tiebreak.

### Reference standings — Tournament I (Winchester, par 70)

| Rank | Player | Points | Strokes |
|------|--------|--------|---------|
| 1 🏆 | Gibbs  | 14     | 88      |
| 2    | Deluca | 9      | 96      |
| 3    | Dennis | 4      | 111     |
| 4    | Angel  | 4      | 115     |
| 5    | E      | 3      | 104     |
| 6    | Franco | 3      | 108     |
| 7    | Adam   | 3      | 110     |
| 8    | Danny  | 2      | 115     |

The Builder must verify their compute function reproduces this table exactly.

## Responsive behaviour

- Desktop (≥1024px): hero full-width, next-tournament + leaderboard side-by-side, scorecard fills the page.
- Tablet (≥640px): single column, scorecard horizontally scrollable inside the card.
- Mobile (<640px): stacked, leaderboard collapses per-tournament finish columns into an expandable row.

## Accessibility

- WCAG AA contrast minimums on all text.
- Keyboard-navigable expand/collapse on tournament cards.
- All non-decorative SVG icons have `aria-hidden="true"` siblings or labels where they convey meaning.

## Out of scope (future)

- Live scoring during a round.
- Photos / image gallery per tournament.
- Player profile pages.
- Authentication / private edit mode.
- Auto-deploy pipeline.
