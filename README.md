<img width="1918" height="907" alt="Screenshot 2026-07-09 120525" src="https://github.com/user-attachments/assets/2157e88e-c8ef-4523-b0e9-9b6be4f6662f" />
# wikipedia · tree

A single-page, single-file web app that turns Wikipedia browsing into an
infinite, explorable tree. Instead of navigating one article at a time, every
link you click spawns a new "card" connected to the article it came from,
so your browsing history becomes a visual, pannable, zoomable mind map.

Open `index.html` in a browser and start clicking.

## How it works

- **Start**: On first load (or after clicking the "new tree" button in the
  top-right), a random Wikipedia article is fetched and shown as the root
  card.
- **Grow the tree**: Click any internal link inside an article card and a
  new child card animates into existence below it, connected by a curved
  edge. Clicking a link that was already opened from that card just refocuses
  the existing child instead of duplicating it.
- **Read in place**: Article HTML is fetched from Wikipedia's REST API and
  sanitized (navigation boxes, references, "See also"/"External links"
  sections, edit links, etc. are stripped) so each card reads like a clean
  article excerpt. External links open in a new tab; in-page anchors scroll
  within the card.
- **Pan & zoom**: Drag the background (or hold `Space` and drag anywhere, or
  middle-click drag) to pan, and scroll/pinch to zoom. `Ctrl/Cmd + 0` fits the
  whole tree in view. A minimap in the bottom-right corner shows your
  position relative to the full tree.
- **Persistence**: The tree's structure and viewport are saved to
  `localStorage`, so reloading the page restores where you left off (article
  content is re-fetched on load).
- **New tree**: The circular button in the top bar clears the current tree
  (with a confirmation) and starts over from a fresh random article.

## Tech

Everything: markup, styling, and logic lives in a single
`index.html` file with no build step and no dependencies. It talks directly
to Wikipedia's public APIs:

- `en.wikipedia.org/w/api.php` (`list=random`) to pick a starting article
- `en.wikipedia.org/api/rest_v1/page/html/...` to fetch article content

Just open the file in any modern browser, no server or install required.
