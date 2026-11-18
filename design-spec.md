# Design Spec — "Paper Edition" (LOCKED)

> Boss-approved palette for the documentation website. **Do not change** without
> explicit new approval from Boss. This file is the single source of design truth.

## Core Palette

| Token      | Value       | Role                                        |
|------------|-------------|---------------------------------------------|
| `--bg`     | `#f7f0df`   | Warm cream background (typewriter paper)    |
| `--paper`  | `#efe6cf`   | Deeper cream for panels / hover / code      |
| `--ink`    | `#1c1a17`   | Matte black body text — plain, no gloss     |
| `--dim`    | `#7a7365`   | Faded gray-brown: stamps, muted labels, em  |
| `--heading`| `#8a3324`   | Burnt brick red — H1/H2/H1-pop, strong, nav active |
| `--green`  | `#2e6e4e`   | Muted forest green — inline code & links    |
| `--line`   | `#d9cdb2`   | Cream hairline borders                      |

## Rules

1. **Background is always cream.** Never dark mode, never blue/gray tints.
2. **Body text is matte black.** No glow, no text-shadow, no neon.
3. **Headings use brick red.** H1 gets a full-width 2px black rule beneath.
4. **Header/footer nav rules:** 2px solid matte black (printed-document feel).
5. **Table headers:** solid matte-black bar with cream text (inverted print style).
6. **Code/links:** forest green is the only decorative color; keep it muted.
7. **Font:** IBM Plex Mono (loaded from Google Fonts), fallback Courier New.
8. **Section markers**: `§` and `###` prefixes rendered in `--dim`, like stamps.
9. **Quotes**: italic, deeper-cream panel, 3px black left edge.
10. **Print-ready**: nav/header hidden on print via `@media print`.

## Rejected Palettes (do not resubmit)

- ❌ CRT amber/brown terminal (v1) — too "dump-like"
- ❌ Neon retro-futurist midnight blue + teal/rose (v2) — Boss: "don't like the futuristic design"

## Implementation

- Live file: `~/nexus-docs/index.html` (served at `http://127.0.0.1:8080/index.html`)
- Backup of rejected v2 kept as `index.v1.html` — reference only, do not serve.
- Renderer: `marked.min.js` (local copy), sidebar SPA fetches `.md` files.

## Status

- ✅ Approved by Boss: "It's nice you have done a good work. Keep this design palette."
