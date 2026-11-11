# kiwimiho.github.io

Personal portfolio site for Summer Shen. Static GitHub Pages site — no build step,
no dependencies. `index.html` is a single self-contained file (all CSS inlined in
the `<style>` block) and is what GitHub Pages serves.

## Files

| File | Role |
|------|------|
| `index.html` | The site. All markup + CSS. Never edit content here first. |
| `content.md` | **Source of truth for all text and links.** The user edits this, and Claude keeps it in sync whenever the page changes. |
| `README.md` | GitHub repo readme only — not the published page. |
| `assets/<project>/` | Screenshots, referenced by relative path. |

## Rule: `content.md` and `index.html` must never drift apart

These two files always describe the same page, and syncing goes **both ways**:

- **Editing `content.md` → update `index.html`.** The normal path (see below).
- **Editing `index.html` directly → update `content.md` in the same turn.** Any
  time text or a link on the page is added, changed, reworded, or removed —
  including quick one-off edits the user asks for in chat, without touching
  `content.md` — mirror the change back into `content.md` before reporting the
  work as done. This is not optional and does not need to be asked for.

What belongs in `content.md`: every piece of visible text and every content link
(project titles, descriptions, project/header links, trailer URLs, tech, roles,
screenshot paths, tab title, meta description). What does not: CSS, colors,
layout, classes, and the Google Fonts `<link>` tags.

Renaming a section, retitling a project, or dropping a link all count as content
changes. Pure styling changes (palette, spacing, animation) do not.

To check nothing drifted, compare the two files' links and assets:

```sh
diff <(grep -o '/file/d/[A-Za-z0-9_-]*' index.html | sort -u) \
     <(grep -o '/file/d/[A-Za-z0-9_-]*' content.md | sort -u)
diff <(grep -o 'assets/[a-z]*/[a-z0-9_]*\.png' index.html | sort -u) \
     <(grep -o 'assets/[a-z]*/[a-z0-9_]*\.png' content.md | sort -u)
```

## Workflow: "update the site from content.md"

`content.md` is the input; `index.html` is the output. When the user asks to update
the site (or edits `content.md` and asks Claude to sync it):

1. Read `content.md` and `index.html`.
2. Diff them section by section and apply **only** the changed content to
   `index.html`. Do not restyle or restructure anything the user didn't change.
3. If a field was **added** to a project that didn't have it before, generate the
   corresponding element using the templates below — matching how that element is
   already written for other projects (same classes, same wording conventions).
4. If a field was **removed**, delete the whole corresponding element (including
   its wrapper `<div class="meta">` or `<p class="shots-label">` if now empty).
5. If a whole `###` project was added, insert a full `<section class="project">`
   in the position it appears in `content.md`, under the right `## Section:` heading.

Section order and project order in `index.html` must always match `content.md`.

## content.md → index.html mapping

| `content.md` | `index.html` |
|--------------|--------------|
| `## Section: X` | `<p class="section-label">X</p>` |
| `### Project Name` | `<h2>` inside `<div class="project-head">` |
| `**Link:** Text — URL` | `<a class="project-link">` next to the `<h2>`, inside `.project-head` |
| `**Description:**` | `<p class="desc">` |
| `**Trailer:** <drive url>` | `<a class="trailer">` block, placed directly after `.desc` |
| `**Trailer thumbnail:**` | overrides the trailer's `<img src>` (see below) |
| `**Tech:** Unity` | `<div><b>Tech</b> <span class="tech-logo">…</span></div>` in `.meta` |
| `**Role:**` | `<div><b>Role</b> <span>…</span></div>` in `.meta` |
| `**Screenshots:**` list | `<p class="shots-label">Screenshots</p>` + `<div class="shots">` |
| `**Name:**` under Header | `<h1>`, all deep green. The surname stays wrapped in `<em>` (which inherits the color) purely as a styling hook. |
| Header `**Links:**` list | `<a>` in `.header-links` |

Fixed element order inside a `<section class="project">`:
**head (title + link) → description → trailer → meta (tech, role) → screenshots.**

## Markup templates

Copy these verbatim, substituting the bracketed parts.

**Project link** (goes inside `.project-head`, after the `<h2>`):

```html
<a class="project-link" href="[URL]" target="_blank" rel="noopener">
  [LINK TEXT]
  <svg viewBox="0 0 12 12" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M4 2h6v6M10 2 2.5 9.5"/></svg>
</a>
```

**Trailer.** Google Drive URLs look like
`https://drive.google.com/file/d/[FILE_ID]/view`. Extract `[FILE_ID]` and use
Drive's thumbnail endpoint for the image, unless the project has a
`**Trailer thumbnail:**` field — then use that local path as the `src` instead.

```html
<a class="trailer" href="https://drive.google.com/file/d/[FILE_ID]/view" target="_blank" rel="noopener">
  <img src="https://drive.google.com/thumbnail?id=[FILE_ID]&sz=w1000" alt="[PROJECT] trailer" loading="lazy">
  <span class="play"><span></span></span>
  <span class="trailer-tag">Trailer</span>
</a>
```

**Tech logo** (logo only — never write the tech's name in text next to it):

```html
<div><b>Tech</b> <span class="tech-logo"><img src="https://upload.wikimedia.org/wikipedia/commons/1/19/Unity_Technologies_logo.svg" alt="Unity"></span></div>
```

**Screenshots:**

```html
<p class="shots-label">Screenshots</p>
<div class="shots">
  <img src="[PATH]" alt="[PROJECT] screenshot" loading="lazy">
</div>
```

**Header link:**

```html
<a href="[URL]" target="_blank" rel="noopener">[TEXT]</a>
```

GitHub's entry also carries an inline `<svg>` logo; plain-text links need no icon.

## Design rules (the user's stated preferences)

- **Sage page, warm ivory cards, green only — no other accent hues.** The page
  (`--bg`) is a clearly green sage and the cards (`--surface`) are a warm ivory;
  that contrast is deliberate, so the project cards visibly stand out against the
  background. Do not lighten the page toward white or cool the cards toward pure
  white — the user asked for both specifically. Accents are all tones of green:
  `--green-deep` for the name, headings, and pill text, `--green` for section
  labels, play buttons, and hover fills, `--green-mid` for pill and tag borders,
  `--green-soft` for accent fills. The user tried a purple accent and rejected it;
  do not reintroduce a second accent color. Palette lives in the `:root` CSS
  variables — change colors there, not inline.
- **The name in `<h1>` is a single color.** The user rejected a two-tone
  first-name/surname split, so `h1 em` uses `color: inherit`.
- **Solid colors only.** No translucency: no `rgba()` fills, no `backdrop-filter`,
  no reduced `opacity` on images, no gradient card backgrounds, no drop shadows.
  Solid fills with 2px borders.
- **Soft shifting background.** `.bg-anim` holds seven blurred drifting blobs
  spread across the green hue range — chartreuse and soft yellow at one end, teal
  and sea green at the other, with leaf/mint/lime between. The user asked for
  visible variation shifted toward blue *and* yellow, so keep both ends present;
  a set of near-identical greens is what they rejected. **But every blob must
  still read as a green.** A sky-blue blob (`#74b6d4`) was tried and rejected for
  looking like "a big blue spot" — the cool end stops at blue-green/teal
  (roughly hue 165–180), never a true blue. Keep them light enough
  that dark text stays readable over the page margins, and keep the page
  background itself sage rather than white. Keep the `prefers-reduced-motion`
  guard.
- **Blob positioning gotcha.** `.bg-anim` is `position: fixed` with
  `inset: -25vh -25vw`, so the layer is only ~150vh tall and does not scroll. A
  blob with `top` beyond ~120vh sits outside the layer and is never visible. Place
  every blob within that band and spread them horizontally instead — the
  background is one static ambient wash for the whole page, not a per-section
  effect.
- **No footer.**
- The Unity logo is dark artwork, so it needs **no CSS filter** on light surfaces.
  A dark theme would need `filter: invert(1) brightness(1.8)` on `.tech-logo img`
  to make it legible again.

### Previous dark palette (kept in case the user wants to switch back)

The site was dark green before the light theme. To restore it, swap these into
`:root`, re-add the `.tech-logo img` invert filter above, set
`<meta name="color-scheme">` to `dark`, darken the five `.bg-anim` blobs to
roughly 15–25% lightness, and flip the few `var(--on-accent)` hover text colors
back to the dark background color.

```css
--bg: #08120e;  --surface: #10201a;  --surface-2: #162c23;
--border: #234436;  --border-lit: #2f6049;
--text: #eaf6f0;  --text-dim: #9fbdae;  --text-mute: #6e8a7c;
--green: #3ddc84;  --green-dim: #1f7a4d;
/* blobs: #124d33, #0e4740, #1a4f1f, #3a5416, #0f4436 */
```
- Every external link gets `target="_blank" rel="noopener"`; images get
  `loading="lazy"` and a real `alt`.

## Previewing

`python3 -m http.server 8788` from the repo root, then open
`http://localhost:8788/index.html`. Relative `assets/` paths require a server —
opening the file directly also works, but serve it to match production.

## Don't invent content

Roles, tech, and links come from `content.md` only. If a project lacks a role,
omit the element — never guess and fill it in.
