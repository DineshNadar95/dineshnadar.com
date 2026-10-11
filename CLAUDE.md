# dineshnadar.com

Personal portfolio site for Dinesh Nadar. The entire site is one self-contained
file, `index.html` (~180 KB, ~1360 lines). There is no build step, no framework,
and no other assets: the resume PDF is embedded as a base64 data URI and all
project mockup screens are inline `<template>` blocks. The only external
resources are Google Fonts (JetBrains Mono, Manrope, IBM Plex Sans, Baloo 2,
Nunito — see the `<link>` tags in `<head>`).

Deployed on Vercel as a static site (framework preset "Other", no build
command, output directory = repo root).

## Layout of index.html

| Lines (approx.) | Content |
|---|---|
| 1–9 | Doctype, `<head>`, Google Fonts links. `<html>` carries `data-look="layers"` and `data-exp="timeline"` |
| 10–372 | All CSS in one `<style>` block, including theme tokens |
| 374–489 | Body markup: `#sky` canvas, hero, tech-stack section `#srun`, experience `#exp`, projects `#prun` (lamp SVG, `#dust` canvas, `#deck`), footer |
| 490–909 | Six `<template>` mockup blocks + the project-detail `#overlay` |
| 911–end | All JS in one `<script>` block: `LINKS`, social rail, sky, `GROUPS` + stack views, `EXP` + experience views, `PROJECTS` + lamp/deck/overlay |

## Data objects (edit these to update content)

### LINKS (~line 912)
`{ resume, linkedin, github, email }`. `resume` is a `data:application/pdf;base64,...`
URI — the whole resume PDF on one very long line. To update the resume, re-encode
the new PDF (`base64 -i resume.pdf`) and replace the data URI. The social rail
(fixed icon nav) is built from LINKS by the "Social links" IIFE just below it;
the resume link downloads as `Dinesh_Nadar_Resume.pdf`.

### GROUPS (~line 972) — tech stack
Array of 5 layers, each `{ name, c, line, items }`:
- `c` is a CSS color token suffix: `fe` (Frontend), `be` (Backend), `ai` (AI),
  `db` (Data), `cl` (Cloud). Each has `--<c>` and `--<c>2` color variables.
- `items` is an array of `[Name, TwoLetterSymbol, description]` triples
  (9 per group).

The stack section `#srun` has **four scroll-driven views**, all rendered up
front from GROUPS and toggled by the `LOOK` variable (default `'layers'`,
matching `data-look` on `<html>`; the `.looks` switcher CSS exists but no
switcher buttons are currently created, so only the default view shows):
- **terminal** (`.v-terminal`): fake terminal typing `dinesh --stack <group>`
  and printing ✓ lines per item.
- **layers** (`.v-layers`) — the 3D "Layers" view: each group becomes a `.slab`
  (a 3×3 grid of `.tile`s plus a `.nm` name label) inside `#scene`, which is
  tilted with `rotateX(58deg) rotateZ(-42deg)`. A rAF loop maps scroll progress
  through the section (one unit per group) to `translateZ` of the scene and of
  each slab, fading non-active slabs; `#isolabel` shows "Layer N of 5", the
  group name, and its `line`.
- **ribbons** (`.v-ribbons`): one horizontal row of words per group, scrolled
  word-by-word with a centered focus word and description.
- **table** (`.v-table`): periodic-table style columns of two-letter symbols
  with a featured element panel.

A screen-reader fallback list of all groups/items is rendered into `#srlists`.

### EXP (~line 1136) — experience timeline, newest first
Array of `{ big, y, to, org, where, role, text, pts }`:
- `big` is the large display year ("Now", "2021", …), `y`/`to` the date range.
- `pts` is the bullet list.
Order: Verizon (current) → Tracflo → Stevens (MS) → Accenture. Keep newest first.

Two views exist, toggled by `EXPV` (fixed at `'timeline'`, mirrored in
`data-exp`): a vertical timeline `#xtimeline` with a progress rail, and a
horizontal station track `#xtrack` (currently unused).

### PROJECTS (~line 1192)
Array of `{ title, status, s, line, tech, lead, how }` plus either:
- `live`: URL → card shows a "Live ↗" tag and the overlay gets an
  "Open the live app" link, and/or
- `mock`: `'wallet'` or `'todo'` → the overlay renders three phone
  frames filled from templates `#mock-<mock>-0/1/2` (see below).
`s` is the accent color (a CSS var reference like `var(--be)`).
All three current projects have `live` URLs: health-rag (CloudFront),
Card wallet (Vercel), Gamified to-do / questlist (Vercel staging).
Current projects: health-rag (live), Card wallet (mock `wallet`), Gamified
to-do (mock `todo`).

## Project detail overlay

Markup at ~line 904: `#overlay` > `.panel` (role=dialog) > `#close` + `#d-body`.
`openProject(i)` builds the panel HTML from the PROJECTS entry with staggered
`.reveal` transition delays, then clones each `<template>` referenced by
`data-mock="mock-<name>-<k>"` into a `.phone > .scr` frame. Escape or clicking
the backdrop closes it; focus is trapped on the close button and returned to
the opening card.

### Mockup templates (lines ~490–909)
Six `<template>` blocks, entirely inline-styled 390×844 px phone screens:
- `#mock-wallet-0/1/2` — Card wallet app (IBM Plex Sans, light gray `#F4F5F7`).
- `#mock-todo-0/1/2` — Gamified to-do app (Nunito, cream `#F5F2EA`).
They are static HTML only (no JS); edit them directly to change the mock screens.

## Projects section: lamp, dust, card shadows

`#prun` is a tall scroll runway (`n * 85 + 60` vh). Inside `#pstage`:
- **Lamp**: `#lampsvg`, a hand-drawn SVG (viewBox 240×340) of a hanging metal
  lamp — cord, brass fittings, brushed-metal shade (gradient + turbulence
  filter `#brush`), glowing bulb and bloom. `layout()` positions it above the
  card deck and derives the light source point (`lightX`, `lightY`).
- **Beam**: `.beam` div, a CSS cone of light below the bulb.
- **Dust**: `#dust` canvas; 70 motes drift down inside the beam cone, drawn
  every frame in `rgba(var(--glow-rgb), a)`, with blend mode `--blend`
  (multiply in light mode, screen in dark).
- **Cards**: one `.pcard` button per project in `#deck`. The rAF loop maps
  scroll progress to a deck shuffle (upcoming cards stack behind/above, passed
  cards fly down-right and fade). Each card gets:
  - `--lit`: how close it is to the light (brightens the card),
  - `--lx`: horizontal light offset for the card's internal gradient,
  - a two-layer `box-shadow` cast **away from the lamp's light point**
    (direction = normalized vector from bulb to card, length/blur grow with
    distance) — this is what sells the lamp effect.
- `#plist` is the numbered side nav; clicking an item scrolls the runway to
  that card.

## Theme (CSS tokens, lines ~11–38)

Light mode is the default `:root` block; dark mode re-declares the same tokens
under `@media (prefers-color-scheme: dark)` guarded by
`:root:not([data-theme="light"])`, and again under `:root[data-theme="dark"]`
(manual override hooks; no JS currently sets `data-theme`, so the site follows
the OS). Tokens:
- Surfaces/text: `--bg`, `--bg2`, `--card`, `--ink`, `--muted`, `--line`
- Effects: `--star` (sky star RGB), `--glow`/`--glow-rgb` (lamp amber),
  `--blend` (dust blend mode), `--shadow-rgb`, `--dark` (0/1 flag)
- Group colors: `--fe/--fe2`, `--be/--be2`, `--ai/--ai2`, `--db/--db2`,
  `--cl/--cl2`
- Fonts: `--mono` (JetBrains Mono), `--sans` (Manrope)

JS reads tokens via `cssv()` and listens to `prefers-color-scheme` changes to
refresh canvas colors.

## Conventions

- Keep everything in index.html; do not split out CSS/JS or add a build step.
- Canvas/scroll animations respect `prefers-reduced-motion` via the `reduce`
  flag — keep new animations behind it.
- All scroll effects use rAF loops gated by IntersectionObserver `visible`
  flags; follow that pattern for anything new.
- When grepping or reading the file, beware the base64 resume line (~170 KB on
  one line) — truncate long lines (`cut -c1-240`) or read by line ranges.
