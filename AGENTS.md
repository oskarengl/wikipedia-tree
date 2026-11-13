# AGENTS.md — Wikipedia Tree

This file is a brief for new Claude agents picking up work on this project.
Read it before making any changes.

## What this is

A single-page web app (`index.html`) that turns Wikipedia browsing into a visual tree.
Every link you click spawns a new article "card" connected to its parent by an animated
curve, so your browsing history becomes a pannable, zoomable mind map.

**How to open it:** Just open `index.html` in a browser — no server, no build step, no
dependencies.

---

## Architecture

Everything — HTML, CSS, JavaScript — lives in one file: `index.html`.
There is no build toolchain, no npm, no imports. Changes go directly in that file.

### CSS (lines ~8–322)
CSS custom properties (`--bg-a`, `--paper`, `--ink`, `--accent`, `--line`, etc.) define
the warm parchment color scheme. All theming flows from those variables on `:root`.

### HTML structure (lines ~324–342)
- `#topbar` — fixed brand label + "new tree" button (`#btn-new`)
- `#version-badge` — fixed bottom-left label (currently `v10 · saw png button`)
- `#viewport` — full-screen container that receives pan/zoom pointer events
  - `#canvas` — absolutely-positioned element that gets CSS `transform: translate() scale()`
    - `#edges` — SVG overlay for connection lines (infinite `overflow: visible`)
    - `.node` elements — appended here by JS at runtime
- `#minimap` — fixed bottom-right 140×100 px overview

### JS (lines ~345–1149)
Divided into clearly-commented sections:

| Section | What it does |
|---------|-------------|
| **Constants** | `NODE_W=480`, `NODE_H_MAX=540`, `HGAP=60`, `VGAP=90` |
| **State** | `state.rootId`, `state.nodes` (map id→node), `view {x,y,scale}`, `focusedId` |
| **Persistence** | `save()` / `load()` using `localStorage` key `wikitree.v2` |
| **Wikipedia** | `fetchRandomTitle()`, `fetchArticleHtml(title)`, `sanitizeArticleHtml(html, nodeId)` |
| **Tree mutation** | `createNode()`, `removeNode()`, `spawnChild()` |
| **DOM** | `buildNodeElement()`, `loadArticleInto()`, `wireLinks()` |
| **Layout** | `subtreeWidth()`, `layoutNode()`, `relayout()` |
| **Edges** | `drawEdges()` — SVG cubic bezier curves |
| **Minimap** | `drawMinimap()` — scaled SVG rects |
| **Pan/zoom** | Wheel, mouse drag, touch events; `applyView()`, `animateView()` |
| **Navigation** | `centerOn(id)`, `fitAll()` (`Ctrl/Cmd+0`) |
| **Init** | `init()` — restores from localStorage or calls `startFresh()` |

### Node object shape
```js
{
  id: string,          // random 8-char alphanumeric
  title: string,       // Wikipedia article title
  parentId: string | null,
  childIds: string[],
  el: HTMLElement,     // the .node div; null before buildNodeElement()
  layout: { x, y, h } // canvas-space position (px)
}
```

---

## Versioning

The version badge (`#version-badge`) is a free-form string the user updates manually.
Current: `v10 · saw png button`. When adding a notable feature, bump the version label
at the user's request or leave it — the user decides.

---

## How the user works with Claude here

The user iterates on features conversationally. Typical requests:
- Add a new interaction feature (keyboard shortcuts, hover effects, etc.)
- Tweak animations (timing, easing, what triggers them)
- Adjust layout constants or CSS variables
- Fix a bug observed in the browser
- Improve Wikipedia content sanitization (things leaking through the filter)

**Testing:** Always test by opening `index.html` in a browser. No automated tests exist.
The browser console is the debugging surface. The user typically confirms visually.

---

## Key gotchas

- **Single-file constraint**: The user wants everything in `index.html`. Do not create
  separate `.js` or `.css` files unless explicitly asked.
- **No dependencies**: Do not add `<script src="...">` CDN imports without asking.
- **Lazy-load / transformed canvas**: Images inside the CSS-transformed `#canvas` don't
  trigger the browser's intersection observer, so Wikipedia images must be force-loaded
  with `loading="eager"` (already handled in `sanitizeArticleHtml`).
- **ID namespacing**: Wikipedia article HTML IDs are prefixed with the node's random ID
  (`nodeId + '__' + el.id`) to prevent cross-node anchor collisions.
- **Edge animation**: During the line-draw animation, the edge's `data-eid` is in
  `animatingEdges` (a `Set`). `drawEdges()` skips creating a static path for it;
  the animated path is managed separately in `spawnChild()`.
- **`localStorage` key**: `wikitree.v2` — if you change the saved schema, bump this.
