# Landing page: add the Windows desktop edition

- **Date:** 2026-08-02
- **Repo:** `live-translator-landing`
- **Scope:** `index.html` only
- **Status:** approved
- **Plan:** [`docs/superpowers/plans/2026-08-02-landing-desktop-edition.md`](../plans/2026-08-02-landing-desktop-edition.md)

## Goal

The landing page presents Live Translator as a browser extension only. A Windows
desktop app now exists and ships from this repo (`desktop/`), but nothing on the
page mentions it. Visitors who need to translate audio from outside the browser
leave without knowing the product covers them.

This change introduces the Windows edition as a first-class option: a second
install path in the header and hero, a comparison section, and guidance on
choosing between the two.

## Non-goals

- The 10 `instruction-*.html` pages. Windows install instructions are a separate
  ticket. This spec only adds a link from the comparison section to the existing
  `instruction.html`.
- `changelog.html`.
- Building or uploading the 0.1.4 installer. That is a manual release step,
  recorded under [Release prerequisite](#release-prerequisite).

## Current state

Relevant facts established while reading `index.html` (1711 lines, single file,
Tailwind via CDN, vanilla JS at the bottom):

- The header CTA is `<a href="#install">` (line 311) and the mobile menu repeats
  it (line 338). **`id="install"` does not exist anywhere in the document** —
  both links are dead today.
- Hero CTA row (line 375) holds a primary "Add to Browser — It's Free" linking to
  the Chrome Web Store, and a ghost "See How It Works" linking to
  `#how-it-works`.
- Hero badge (line 363) reads "Free & Private Browser Extension".
- Section order: hero → `#demo-video` → `#whats-new` → trust bar → `#features` →
  `#pricing` → `#platforms` → `#how-it-works` → `#narration-guide` → privacy →
  `#faq` → footer.
- `#platforms` (line 1053) is a `grid sm:grid-cols-2 lg:grid-cols-5` with 12
  cards, ending in "Any Website".
- `desktop/` already hosts `LiveTranslator-Setup-0.1.1.exe`,
  `LiveTranslator-Setup-0.1.2.exe` and `latest.yml` (the electron-builder
  auto-update feed). The desktop repo is at version **0.1.4**.
- `gtag` is configured (`G-T7YFE7XLLF`) and available globally.
- The mobile menu toggle at the bottom of the file uses
  `classList.toggle('hidden')` — the pattern to reuse for the new dropdown.

## Feature matrix (source of truth)

Derived from the `live-translator` repo, and confirmed with the product owner.
Desktop has no narration/TTS, no webpage translate, no screen recording, no
Notion/browser auto-save, no context presets, and no custom speaker names.

| Feature | Extension | Windows |
| --- | --- | --- |
| Real-time transcription + translation (60+ languages) | Yes | Yes |
| Audio source | Browser tab + mic | **Any app on your PC** (system audio) + mic |
| Subtitle overlay | Yes | Yes (always-on-top window) |
| AI Assistant | Yes | Yes |
| Bring your own key (Soniox / OpenAI) | Yes | Yes |
| Translate webpage | Yes | — |
| Narrate Their Voice (dubbing) | Yes | — |
| Narrate My Voice (two-way translate) | Yes | — |
| Screen recording | Yes | — |
| Auto-save to Browser / Notion | Yes | — |
| Context settings + custom speaker names | Yes | — |
| Installation required | No | Yes (~96 MB installer) |
| Platform | Chrome, Edge, Brave | Windows 10/11 |

A row comparing meeting-site integration (Meet/Zoom/Teams/YouTube) was
deliberately dropped: the desktop app reaches those same platforms through system
audio, so the row would mislead.

## Design

### 1. Header — "Install Free" becomes a dropdown

Replace the dead `<a href="#install">` with a button that toggles a glass panel
containing two entries:

```
[ ⬇ Install Free ▾ ]
   ┌──────────────────────────────────┐
   │ 🌐 Browser Extension             │
   │    Chrome, Edge, Brave — free    │
   │ ⊞ Windows App                    │
   │    Windows 10/11 · v0.1.4        │
   └──────────────────────────────────┘
```

- Panel is absolutely positioned under the button, right-aligned, using the
  existing `glass` / `border-white/10` / `rounded-2xl` classes.
- Toggle with `classList.toggle('hidden')`, mirroring the existing mobile-menu
  handler. Close on outside click and on `Escape`.
- Accessibility: `aria-expanded` on the button, `aria-haspopup="menu"` and
  `role="menu"` on the panel, `role="menuitem"` on the two links. Keep them
  focusable in DOM order.
- The mobile menu does **not** get a dropdown. Its single "Install Free" button
  splits into two stacked buttons (Browser Extension, Windows App) — the menu is
  already expanded, so nesting a disclosure inside it adds nothing.

### 2. Hero

Current: `[Add to Browser — It's Free]` `[See How It Works]`.

New: `[Add to Browser — It's Free]` `[⊞ Get Windows App]`, with
"See How It Works ↓" demoted to a small text link below the button row.

Rationale: three buttons on one row dilutes the primary CTA. Windows is a real
install intent; "See How It Works" is not.

Style the Windows button as a secondary/outlined variant (`border-white/10`,
`hover:bg-white/5`) so the Chrome Web Store button stays visually dominant.
Include a Windows logo glyph (four-pane SVG) and a sub-caption
`Windows 10/11 · v0.1.4 · ~96 MB`.

Badge copy changes from "Free & Private Browser Extension" to
"Free & Private — Extension + Windows App".

The third trust badge ("Works on Chrome, Edge, Brave") becomes
"Chrome, Edge, Brave + Windows app".

### 3. New section `#compare`, between `#features` and `#pricing`

Placed so the reader learns what the product does, picks an edition, then sees
the price — and so the "one license covers both" note lands immediately before
the pricing cards.

**3a. Comparison table.** The 13 rows above, rendered as a real `<table>` inside
an `overflow-x-auto` wrapper. Feature column is `sticky left-0` with the section
background so it stays readable while scrolling on narrow screens. Column headers
carry the same icons used in the header dropdown. Yes/no cells use the existing
emerald check SVG and a muted em-dash. `<caption class="sr-only">` describes the
table for screen readers.

Do not restructure the table into cards on mobile. Horizontal scroll is the
smaller, more maintainable answer, and the row labels stay comparable.

**3b. "Which should I choose?"** — two `glass-card` panels below the table:

- **Choose the Extension if…** you meet and watch inside your browser and want
  every feature. Most capable version, nothing to install. Limitation: it can
  only translate audio playing in the browser.
- **Choose Windows if…** you need to translate audio from *any* application —
  the Zoom or Teams desktop client, a media player, a game, internal software.
  Fewer features, but no restriction on where the sound comes from.

Below the two panels, an accent callout: **You can install both.** Same account,
same license — use whichever fits the moment.

Each panel ends with its own install button (Chrome Web Store / direct download),
so the section is self-sufficient. Under the Windows panel, a quiet link:
"Need install help? See the user guide" → `./instruction.html`.

### 4. Download link and versioning

All Windows install links point to a **fixed filename**:

```
./desktop/LiveTranslator-Setup-latest.exe
```

The HTML never changes across releases. Version, platform and size
(`v0.1.4 · Windows 10/11 · ~96 MB`) are hardcoded in the markup — a `fetch` of
`latest.yml` would add a network request and JS purely to render one number.

Accept that the hardcoded version string drifts if a release forgets to update
it; the download itself never breaks, which is the part that matters.

#### Release prerequisite

Each desktop release must upload the installer under **two** names in `desktop/`:

- `LiveTranslator-Setup-<version>.exe` — referenced by `latest.yml`, required for
  electron-builder auto-update.
- `LiveTranslator-Setup-latest.exe` — referenced by the landing page.

For this change, 0.1.4 must be built and uploaded under both names before the
page ships. The product owner handles this.

### 5. Analytics

Add `gtag('event', 'download_windows', { edition: 'windows', location: '<where>' })`
to every Windows download link, with `location` distinguishing `header`,
`hero`, `compare` and `pricing`. One inline handler shared by all four via a
`data-download-edition` attribute and a single delegated listener, so adding a
fifth link later needs no JS change.

### 6. Pricing section

Add one line under the plan grid: **Every plan covers both the browser extension
and the Windows app.** Prevents the reading that each edition is billed
separately.

### 7. `#platforms` section

Append a 13th card after "Any Website": **Any Windows App** — "Zoom and Teams
desktop clients, media players, games, internal tools — anything that plays
sound, with the Windows app."

The grid is `lg:grid-cols-5` with 12 cards today (last row holds 2); a 13th makes
it 3. No layout change needed.

Section subtitle gains a trailing clause acknowledging the desktop app so the
list does not imply the browser is the only reach.

### 8. FAQ

Two new items appended to `#faq-container`, matching the existing
`faq-item glass-card` markup:

- **What is the difference between the extension and the Windows app?** — short
  answer plus a link to `#compare`.
- **Does my Pro license work on both?** — yes, one account and one license cover
  both; you may run them side by side.

### 9. Meta / SEO

- `<title>`: unchanged (already edition-neutral).
- `<meta name="description">`, `og:description`, `twitter:description`: replace
  the "is a browser extension" framing with "browser extension and Windows app".
- `<meta name="keywords">`: add `windows translator app`,
  `desktop translation app`, `system audio translation`.
- JSON-LD `SoftwareApplication`: `applicationCategory` becomes
  `["BrowserApplication", "DesktopApplication"]`; `operatingSystem` becomes
  `"Chrome, Edge, Brave, Arc, Windows"`; `description` mirrors the new meta
  description; `featureList` gains "Windows desktop app that translates audio
  from any application on your PC".

### 10. Navigation

Both the desktop nav and the mobile menu gain a `Compare` link to `#compare`,
inserted after `Features`. The desktop nav goes from 7 to 8 links — acceptable,
but check wrapping at the `md` breakpoint.

### 11. Fixed in passing

The dead `#install` anchor disappears: the header CTA becomes a dropdown with two
real destinations, and the mobile menu link is replaced by two real links.

## Acceptance criteria

1. No link in `index.html` points at `#install`; no dead in-page anchors remain.
2. Header "Install Free" opens a two-item dropdown; it closes on outside click
   and on `Escape`; `aria-expanded` reflects state.
3. Hero shows two buttons (Chrome Web Store, Windows download) with
   "See How It Works" as a text link below.
4. `#compare` exists between `#features` and `#pricing`, contains the 13-row
   table, two guidance panels, and the "install both" callout.
5. Every Windows download link resolves to
   `./desktop/LiveTranslator-Setup-latest.exe`.
6. Clicking any Windows download link fires a `download_windows` gtag event
   carrying a `location` value.
7. Pricing section states plans cover both editions.
8. `#platforms` includes an "Any Windows App" card.
9. Two new FAQ entries render and expand using the existing accordion script.
10. Meta description, OG/Twitter descriptions, keywords and JSON-LD all mention
    the Windows app; JSON-LD remains valid (Rich Results Test passes).
11. Desktop nav and mobile menu both contain a `Compare` link; the desktop nav
    does not wrap at the `md` breakpoint.
12. At 375 px width: the comparison table scrolls horizontally, the page body
    does not; the mobile menu shows two install buttons.
13. No new external dependency; Tailwind CDN and vanilla JS only.

## Verification

Manual, since the repo has no test harness:

- Open `index.html` in a browser at 1440 px and 375 px.
- Click through every install link and confirm the destination.
- Confirm the download link returns the file (once 0.1.4 is uploaded).
- Keyboard-only pass over the header dropdown: Tab to it, Enter to open, Tab
  through both items, Escape to close.
- Confirm the four gtag events fire (DevTools network filter on
  `google-analytics`, or `window.dataLayer` inspection).
