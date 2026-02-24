# CLAUDE.md — ぴすた / Pista Personal Link Hub

This file provides guidance for AI assistants (Claude and others) working on this codebase.

## Project Overview

A zero-dependency personal link hub (linktree-style) for a Japanese running enthusiast. Built with pure HTML, CSS, and vanilla JavaScript — no frameworks, no build tools, no package manager.

**Live site**: https://pista-run-dev.github.io/mypage/
**Deployment**: GitHub Pages (push to `main` → auto-deployed)

---

## Repository Structure

```
mypage/
├── index.html        # Single-page app entry point (Japanese, lang="ja")
├── style.css         # All styles — design tokens, themes, layout, components
├── sw.js             # Service Worker for offline/PWA caching
├── manifest.json     # PWA manifest (installable app metadata)
├── avatar.jpg        # Profile photo
├── icons/
│   ├── apple-touch-icon.png   # iOS home screen icon
│   ├── icon-192.png           # Android/PWA icon
│   └── icon-512.png           # Android/PWA icon (also used as maskable)
└── README.md
```

There is **no build step**. Files are served directly as static assets. Do not introduce a build pipeline unless explicitly requested.

---

## Architecture & Key Patterns

### Theme System

The site uses a CSS attribute-based theme system:

- `<html data-theme="dark">` or `<html data-theme="light">` — default is `dark`
- CSS variables are scoped to `[data-theme="dark"]` and `[data-theme="light"]` blocks in `style.css`
- Theme preference is persisted via `localStorage.getItem('theme')` / `localStorage.setItem('theme', ...)`
- The browser's `<meta name="theme-color">` tag (id=`meta-theme-color`) is updated on toggle: dark → `#050510`, light → `#f2f2f7`
- The theme toggle button is fixed bottom-right (`position: fixed; bottom: ...; right: 20px`)

**Do not** use `prefers-color-scheme` media queries — the site relies exclusively on the `data-theme` attribute.

### CSS Design Tokens

All shared values are CSS custom properties defined in `:root` (`style.css:15–36`):

| Variable | Value | Purpose |
|---|---|---|
| `--gap` | `10px` | Grid/card spacing |
| `--r-card` | `22px` (mobile), `26px` (≥600px) | Card border radius |
| `--r-icon` | `12px` | Brand icon badge radius |
| `--max-w` | `480px` | Max content width |
| `--glass-blur` | `16px` | Backdrop blur for glassmorphism |
| `--blue` | `#0a84ff` | Primary accent |
| `--purple` | `#bf5af2` | Secondary accent |
| `--teal` | `#64d2ff` | Tertiary accent |

Theme-scoped variables (`--bg`, `--surface`, `--text`, `--text-2`, `--text-3`, `--border`, `--card-shadow`, `--card-inset`, `--mesh-opacity`) are defined separately inside `[data-theme="dark"]` and `[data-theme="light"]` blocks.

**Always use these variables** — never hardcode hex colors in new rules unless adding a new brand color constant to `:root`.

### Bento Grid Layout

The main layout uses CSS Grid with named template areas (`style.css:136–150`):

**Mobile (default, 2 columns):**
```
profile   profile
instagram x
note      runtrip
strava    youtube
spotify   spotify
teaser    teaser
```

**Tablet ≥600px (3 columns):**
```
profile   profile   profile
instagram x         note
runtrip   strava    youtube
spotify   teaser    teaser
```

Each card is assigned its grid area via `.card--{platform}` class. If adding a new card, assign it a named grid area and update both `grid-template-areas` blocks.

### Glassmorphism Card Pattern

All cards share the `.card` base class (`style.css:152–163`) providing:
- `background: var(--surface)` — semi-transparent surface
- `backdrop-filter: blur(var(--glass-blur))` with `-webkit-` prefix
- `border: 1px solid var(--border)`
- `box-shadow: var(--card-shadow), var(--card-inset)` — outer shadow + inset highlight
- Staggered entrance animation via `cardEnter` keyframe + `:nth-child()` delays (0.08s increments)

### Card Component Types

1. **`.card--profile`** — non-interactive, displays avatar + bio + personal bests
2. **`.card--link`** — `<a>` tag, hover/active states, external link to a platform
3. **`.card--teaser`** — non-interactive, promotes upcoming app with animated badge

### Link Card Structure

```html
<a class="card card--link card--{platform}"
   href="..."
   target="_blank"
   rel="noopener noreferrer"
   aria-label="{Platform} を開く">
  <div class="link-top">
    <span class="brand-icon">
      <svg ...></svg>
    </span>
    <span class="link-arrow" aria-hidden="true">↗</span>
  </div>
  <h2 class="link-name">{Platform Name}</h2>
  <p class="link-desc">{Japanese description}</p>
</a>
```

### Brand Icon Colors

Each platform has a color variable in `:root` and matching `.card--{platform} .brand-icon` rules. Brand-specific hover glow effects are defined separately for dark and light themes using `[data-theme="dark/light"] .card--{platform}:hover` selectors.

Current platforms and their color variables:
- `--c-instagram` — gradient (135deg, orange → pink → magenta)
- `--c-x` — `#000` (black, with border)
- `--c-note` — `#41c9b4` (teal)
- `--c-runtrip` — `#3AB483` (green); uses white background icon badge
- `--c-strava` — `#FC4C02` (orange)
- `--c-youtube` — `#FF0000` (red)
- `--c-spotify` — `#1DB954` (green)

### Animations

| Keyframe | Duration | Purpose |
|---|---|---|
| `cardEnter` | 0.6s | Card entrance (translateY + scale, staggered) |
| `meshShift` | 12s infinite alternate | Background mesh gradient movement |
| `blink` | 2s infinite | Teaser badge pulsing dot |

Easing convention: `cubic-bezier(0.22, 1, 0.36, 1)` for motion, `ease` for color/opacity transitions.

### Service Worker (sw.js)

Cache name: `pista-v1`. Strategy: **cache-first with background revalidation** (stale-while-revalidate pattern).

- `install`: pre-caches `/mypage/`, `index.html`, `style.css`, `manifest.json`, and both icons
- `activate`: deletes all caches except the current `CACHE_NAME`
- `fetch`: serves from cache immediately if available, simultaneously fetches from network and updates cache

**When adding new assets** that should work offline, add their paths to the `ASSETS` array in `sw.js` and increment the cache version (`pista-v1` → `pista-v2`) to force re-caching.

---

## CSS Naming Conventions

- **Block**: `.card`, `.bento`, `.footer`, `.theme-toggle`
- **Modifier** (double dash): `.card--profile`, `.card--link`, `.card--teaser`, `.card--instagram`
- **Element** (double underscore): `.pb__label`, `.pb__time`, `.teaser-badge__dot`
- **State/utility**: `.dot--active`
- All class names are kebab-case

---

## HTML Conventions

- Language: `lang="ja"` (Japanese)
- Default theme: `data-theme="dark"` on `<html>`
- All external links must have `target="_blank"` and `rel="noopener noreferrer"`
- All interactive elements need `aria-label` in Japanese (e.g., `aria-label="Instagram を開く"`)
- Decorative SVG icons use `aria-hidden="true"`
- Heading hierarchy: `<h1>` for the profile name, `<h2>` for card titles
- SVG icons are inlined (no external icon library)

---

## JavaScript Conventions

- Vanilla JS only — no libraries or frameworks
- The theme script uses an IIFE pattern and runs after `</body>` (bottom of file)
- Service Worker registration is a simple inline script before `</body>`
- Direct DOM manipulation via `getElementById` / `setAttribute`
- No module system — no `import`/`export`

---

## Responsive Breakpoints

| Breakpoint | Behavior |
|---|---|
| Default (mobile) | 2-column bento grid, `--r-card: 22px` |
| `min-width: 600px` | 3-column bento grid, `--r-card: 26px` |
| `max-width: 340px` | Profile card stacks vertically, name wraps |

Uses `100svh` (safe viewport height) and `env(safe-area-inset-bottom)` for notch/home-bar safe areas.

---

## Deployment

Push to `main` branch → GitHub Pages auto-deploys.

- No build step required
- No CI/CD pipeline — deployment is direct
- The PWA scope is `/mypage/` — all asset paths in `sw.js` and `manifest.json` must be prefixed accordingly

---

## Adding a New Social Card

1. **CSS**: Add `--c-{platform}: ...` color variable to `:root` in `style.css`
2. **CSS**: Add `.card--{platform} .brand-icon { background: var(--c-{platform}); }` rule
3. **CSS**: Add hover glow rules for dark and light themes
4. **CSS**: Add `.card--{platform} { grid-area: {platform}; }` rule
5. **CSS**: Add `{platform}` to both `grid-template-areas` blocks (mobile and ≥600px)
6. **HTML**: Add the `<a class="card card--link card--{platform}">` block with correct structure
7. **sw.js**: No changes needed unless adding a new asset file

---

## Known Issues / Notes

- The YouTube and Spotify cards currently link to `href="#"` (placeholders, not yet connected to real profiles)
- There are two Strava cards in the HTML (`index.html:101` and `index.html:160`) — the second appears to be a duplicate that should be reconciled with the intended grid layout
- The grid template defines a `spotify` area but the second Strava card occupies what may be intended as the `strava` area — verify intended layout before editing cards
- Commits are GPG-signed via SSH key (`/home/claude/.ssh/commit_signing_key.pub`)
