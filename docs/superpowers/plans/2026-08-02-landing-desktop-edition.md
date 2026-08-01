# Landing Page — Windows Desktop Edition Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Spec:** [`docs/superpowers/specs/2026-08-02-landing-desktop-edition-design.md`](../specs/2026-08-02-landing-desktop-edition-design.md)

**Goal:** Introduce the Windows desktop app as a first-class install option on the Live Translator landing page — header dropdown, hero CTA, a comparison section, and supporting copy/SEO — entirely within `index.html`.

**Architecture:** `index.html` is a single 1711-line static file: Tailwind from CDN, one `<style>` block, one `<script>` block at the bottom holding all vanilla JS. Every change is an in-place edit to that file. New JS goes into the existing bottom `<script>` block; new markup reuses the existing `glass`, `glass-card`, `btn-primary`, `reveal` and `section-divider` classes. No build step, no new files, no dependencies.

**Tech Stack:** Static HTML5, Tailwind CSS via `cdn.tailwindcss.com`, vanilla ES2020 JS, Google Analytics `gtag`.

## Global Constraints

- **Repository:** `live-translator-landing` (separate from `live-translator`). All work happens in `E:\ezitech\live-translator-landing`.
- **Files touched:** `index.html` only. Do not modify `instruction-*.html`, `changelog.html`, `privacy-policy.html`, or anything under `desktop/`.
- **No new dependencies.** No npm, no build step, no additional `<script src>` or `<link href>` to any external host.
- **Windows download URL is exactly:** `./desktop/LiveTranslator-Setup-latest.exe` — used verbatim by every Windows install link. Never link a versioned filename.
- **Chrome Web Store URL is exactly:** `https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh`
- **Version string shown to users is exactly:** `v0.1.4` — hardcoded, appearing alongside `Windows 10/11` and `~96 MB`.
- **Download-tracking convention:** every install link (both editions) carries `data-download-edition="windows"` or `data-download-edition="extension"` plus `data-download-location="<header|hero|compare|pricing>"`. A single delegated listener (added in Task 1) reads those attributes. Never attach a per-link `onclick`.
- **Do not use the anchor `#install`.** It is dead today and is being removed. Any new in-page anchor must point at a section id that exists.
- **Copy rule:** the extension is never called "the only" or "the" way to use Live Translator. Where existing copy says "browser extension" as the whole product, it becomes "browser extension and Windows app".
- **Reveal animation:** any new top-level block that should animate in gets `class="... reveal"`; the existing `IntersectionObserver` picks it up with no JS change.
- **Line numbers in this plan refer to the file as it stands before Task 1.** They shift as tasks land — always locate edit sites by the quoted snippet, not the line number.

## Testing approach

This repo has no test harness, no package.json, and no build step — and this plan does not add one. Adding Vitest/Playwright to a single static HTML file to assert that a `<section>` exists would be more machinery than the thing it guards.

Each task instead verifies with:

1. **`grep` assertions** — exact, runnable, catch typos and missing attributes.
2. **A browser check** — open `index.html` directly (`file://`) and confirm the described behavior.

Run grep commands from the repo root in Git Bash.

---

### Task 1: Header — install dropdown, nav link, mobile install buttons, download tracking

Replaces the dead `#install` CTA with a real two-destination dropdown, adds the `Compare` nav link, splits the mobile install button in two, and establishes the download-tracking listener that every later task relies on.

**Files:**
- Modify: `index.html` — nav block (~lines 298–344), bottom `<script>` block (~lines 1666–1680)

**Interfaces:**
- Consumes: nothing (first task).
- Produces:
  - Element ids `install-menu-btn` and `install-menu` (header dropdown button + panel).
  - The `data-download-edition` / `data-download-location` attribute convention and its delegated click listener. Every later task that adds an install link sets both attributes and needs no JS.
  - Section anchor `#compare` is *referenced* here by the nav link; Task 3 creates the section. Between Task 1 and Task 3 that nav link scrolls nowhere — expected, resolved in Task 3.

- [ ] **Step 1: Add the `Compare` link to the desktop nav**

Find this line in the desktop nav block:

```html
          <a href="#features" class="font-body text-sm text-slate-400 hover:text-white transition-colors duration-200">Features</a>
```

Insert immediately after it:

```html
          <a href="#compare" class="font-body text-sm text-slate-400 hover:text-white transition-colors duration-200">Compare</a>
```

- [ ] **Step 2: Replace the header CTA with the install dropdown**

Find this exact block:

```html
        <!-- CTA -->
        <div class="hidden md:flex items-center gap-4">
          <a href="#install" class="btn-primary relative inline-flex items-center gap-2 px-5 py-2.5 rounded-xl font-display font-semibold text-sm text-white">
            <span class="relative z-10 flex items-center gap-2">
              <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" /></svg>
              Install Free
            </span>
          </a>
        </div>
```

Replace it with:

```html
        <!-- CTA -->
        <div class="hidden md:flex items-center gap-4 relative">
          <button id="install-menu-btn" type="button" aria-haspopup="menu" aria-expanded="false" aria-controls="install-menu" class="btn-primary relative inline-flex items-center gap-2 px-5 py-2.5 rounded-xl font-display font-semibold text-sm text-white">
            <span class="relative z-10 flex items-center gap-2">
              <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" /></svg>
              Install Free
              <svg class="w-3.5 h-3.5 opacity-70" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2.5"><path stroke-linecap="round" stroke-linejoin="round" d="M19 9l-7 7-7-7" /></svg>
            </span>
          </button>

          <div id="install-menu" role="menu" aria-labelledby="install-menu-btn" class="hidden absolute right-0 top-full mt-3 w-72 glass rounded-2xl border border-white/10 shadow-2xl shadow-black/50 p-2 z-50">
            <a role="menuitem" href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" data-download-edition="extension" data-download-location="header" class="flex items-start gap-3 p-3 rounded-xl hover:bg-white/5 transition-colors">
              <span class="w-9 h-9 rounded-lg bg-accent-blue/10 border border-accent-blue/20 flex items-center justify-center flex-shrink-0">
                <svg class="text-accent-blue" viewBox="0 0 24 24" fill="currentColor" style="width:1.125rem;height:1.125rem"><path d="M12 0C8.21 0 4.831 1.757 2.632 4.501l3.953 3.953A7.48 7.48 0 0112 4.5c1.77 0 3.397.617 4.682 1.645l3.953-3.953C18.435 1.083 15.382 0 12 0zm0 24c3.79 0 7.169-1.757 9.368-4.501l-3.953-3.953A7.48 7.48 0 0112 19.5a7.48 7.48 0 01-5.415-3.954l-3.953 3.953C5.565 22.243 8.618 24 12 24zM1.757 7.632A11.946 11.946 0 000 12c0 1.58.307 3.09.864 4.472l4.812-2.779A7.444 7.444 0 014.5 12c0-.592.069-1.169.198-1.723L1.757 7.632zm20.486 0l-2.941 2.645c.129.554.198 1.131.198 1.723s-.069 1.169-.198 1.723l2.941 2.645A11.946 11.946 0 0024 12c0-1.58-.307-3.09-.864-4.472z"/></svg>
              </span>
              <span class="min-w-0">
                <span class="block font-display font-semibold text-sm text-white">Browser Extension</span>
                <span class="block font-body text-xs text-slate-500 mt-0.5">Chrome, Edge, Brave — free</span>
              </span>
            </a>
            <a role="menuitem" href="./desktop/LiveTranslator-Setup-latest.exe" data-download-edition="windows" data-download-location="header" class="flex items-start gap-3 p-3 rounded-xl hover:bg-white/5 transition-colors">
              <span class="w-9 h-9 rounded-lg bg-accent-cyan/10 border border-accent-cyan/20 flex items-center justify-center flex-shrink-0">
                <svg class="text-accent-cyan" viewBox="0 0 24 24" fill="currentColor" style="width:1.125rem;height:1.125rem"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
              </span>
              <span class="min-w-0">
                <span class="block font-display font-semibold text-sm text-white">Windows App</span>
                <span class="block font-body text-xs text-slate-500 mt-0.5">Windows 10/11 · v0.1.4</span>
              </span>
            </a>
          </div>
        </div>
```

- [ ] **Step 3: Update the mobile menu**

Find this line in the mobile nav block:

```html
          <a href="#features" class="font-body text-sm text-slate-400 hover:text-white transition-colors">Features</a>
```

Insert immediately after it:

```html
          <a href="#compare" class="font-body text-sm text-slate-400 hover:text-white transition-colors">Compare</a>
```

Then find this block in the same mobile nav:

```html
          <a href="#install" class="btn-primary relative inline-flex items-center justify-center gap-2 px-5 py-2.5 rounded-xl font-display font-semibold text-sm text-white mt-2">
            <span class="relative z-10">Install Free</span>
          </a>
```

Replace it with:

```html
          <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" data-download-edition="extension" data-download-location="header" class="btn-primary relative inline-flex items-center justify-center gap-2 px-5 py-2.5 rounded-xl font-display font-semibold text-sm text-white mt-2">
            <span class="relative z-10">Install Browser Extension</span>
          </a>
          <a href="./desktop/LiveTranslator-Setup-latest.exe" data-download-edition="windows" data-download-location="header" class="inline-flex items-center justify-center gap-2 px-5 py-2.5 rounded-xl font-display font-semibold text-sm text-slate-300 border border-white/10 hover:border-white/20 hover:text-white transition-all">
            <svg viewBox="0 0 24 24" fill="currentColor" style="width:1rem;height:1rem"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
            Download for Windows
          </a>
```

Note: the existing mobile-menu script closes the menu on any `<a>` click. Both new links are anchors, so they inherit that behavior — no JS change needed.

- [ ] **Step 4: Add the dropdown and tracking JS**

In the bottom `<script>` block, find:

```js
    mobileMenu.querySelectorAll('a').forEach(link => {
      link.addEventListener('click', () => {
        mobileMenu.classList.add('hidden');
      });
    });
```

Insert immediately after it:

```js
    // Install dropdown (header)
    const installBtn = document.getElementById('install-menu-btn');
    const installMenu = document.getElementById('install-menu');

    function closeInstallMenu() {
      installMenu.classList.add('hidden');
      installBtn.setAttribute('aria-expanded', 'false');
    }

    installBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      const isHidden = installMenu.classList.toggle('hidden');
      installBtn.setAttribute('aria-expanded', String(!isHidden));
    });

    document.addEventListener('click', (e) => {
      if (installMenu.classList.contains('hidden')) return;
      if (installMenu.contains(e.target)) return;
      closeInstallMenu();
    });

    document.addEventListener('keydown', (e) => {
      if (e.key !== 'Escape' || installMenu.classList.contains('hidden')) return;
      closeInstallMenu();
      installBtn.focus();
    });

    // Install download tracking
    document.addEventListener('click', (e) => {
      const link = e.target.closest('[data-download-edition]');
      if (!link || typeof gtag !== 'function') return;
      gtag('event', 'download_' + link.dataset.downloadEdition, {
        edition: link.dataset.downloadEdition,
        location: link.dataset.downloadLocation
      });
    });
```

`classList.toggle` returns `true` when the class was **added** — i.e. when the menu just became hidden. Hence `aria-expanded` is set to `!isHidden`.

- [ ] **Step 5: Verify with grep**

```bash
cd /e/ezitech/live-translator-landing && grep -c '#install"' index.html; grep -c 'id="install-menu"' index.html; grep -c 'data-download-edition' index.html; grep -c 'href="#compare"' index.html
```

Expected output, in order: `0` (grep exits 1 and prints 0 — the dead anchor is gone), `1`, `4`, `2`.

- [ ] **Step 6: Verify in a browser**

Open `index.html` at a viewport ≥ 1024 px:

1. Click "Install Free" — panel opens with two entries.
2. Click anywhere outside — panel closes.
3. Reopen, press `Escape` — panel closes and focus returns to the button.
4. Inspect the button: `aria-expanded` flips between `"true"` and `"false"`.
5. Confirm the desktop nav still fits on one line at 1280 px (8 links now).
6. Resize to 375 px, open the hamburger menu — two install buttons stacked, plus a `Compare` link.
7. In DevTools console, run `dataLayer.length`, click the Windows entry, then re-run — the value increased.

- [ ] **Step 7: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "feat(landing): install dropdown with browser and windows editions"
```

---

### Task 2: Hero — Windows CTA, badge and trust-badge copy

**Files:**
- Modify: `index.html` — hero CTA row (~lines 374–402), hero badge (~line 361)

**Interfaces:**
- Consumes: the `data-download-edition` / `data-download-location` convention from Task 1.
- Produces: nothing later tasks depend on.

- [ ] **Step 1: Update the hero badge copy**

Find:

```html
            <span class="font-body text-xs font-medium text-slate-300 tracking-wide">Free & Private Browser Extension</span>
```

Replace with:

```html
            <span class="font-body text-xs font-medium text-slate-300 tracking-wide">Free & Private — Extension + Windows App</span>
```

- [ ] **Step 2: Replace the hero CTA row**

Find this exact block:

```html
            <a href="#how-it-works" class="inline-flex items-center justify-center gap-2 px-7 py-3.5 rounded-2xl font-display font-semibold text-sm text-slate-300 border border-white/10 hover:border-white/20 hover:text-white transition-all duration-300 hover:bg-white/5">
              See How It Works
              <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M19 9l-7 7-7-7" /></svg>
            </a>
          </div>
```

Replace it with:

```html
            <a href="./desktop/LiveTranslator-Setup-latest.exe" data-download-edition="windows" data-download-location="hero" class="group inline-flex flex-col items-center justify-center px-7 py-3 rounded-2xl border border-white/10 hover:border-white/25 transition-all duration-300 hover:bg-white/5">
              <span class="inline-flex items-center gap-2.5 font-display font-bold text-base text-slate-200 group-hover:text-white transition-colors">
                <svg viewBox="0 0 24 24" fill="currentColor" style="width:1.125rem;height:1.125rem"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
                Get Windows App
              </span>
              <span class="font-body text-[11px] text-slate-500 mt-0.5">Windows 10/11 · v0.1.4 · ~96 MB</span>
            </a>
          </div>

          <a href="#how-it-works" class="inline-flex items-center gap-1.5 mt-5 font-body text-sm text-slate-500 hover:text-slate-300 transition-colors" style="animation: fadeIn 0.8s ease-out 0.5s forwards; opacity: 0;">
            See How It Works
            <svg class="w-3.5 h-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M19 9l-7 7-7-7" /></svg>
          </a>
```

- [ ] **Step 3: Add the tracking attributes to the existing Chrome Web Store hero button**

Find (still in the hero, the button immediately above the one just replaced):

```html
            <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" class="btn-primary relative inline-flex items-center justify-center gap-2.5 px-7 py-3.5 rounded-2xl font-display font-bold text-base text-white shadow-xl shadow-accent-blue/20">
```

Replace with:

```html
            <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" data-download-edition="extension" data-download-location="hero" class="btn-primary relative inline-flex items-center justify-center gap-2.5 px-7 py-3.5 rounded-2xl font-display font-bold text-base text-white shadow-xl shadow-accent-blue/20">
```

- [ ] **Step 4: Update the third trust badge**

Find:

```html
              <span class="font-body text-xs">Works on Chrome, Edge, Brave</span>
```

Replace with:

```html
              <span class="font-body text-xs">Chrome, Edge, Brave + Windows app</span>
```

- [ ] **Step 5: Verify with grep**

```bash
cd /e/ezitech/live-translator-landing && grep -c 'data-download-location="hero"' index.html; grep -c 'Get Windows App' index.html; grep -c 'Free & Private — Extension' index.html
```

Expected: `2`, `1`, `1`.

- [ ] **Step 6: Verify in a browser**

1. At 1440 px: two buttons side by side; the blue Chrome button is visually dominant; "See How It Works ↓" sits below as small grey text.
2. At 375 px: buttons stack; neither overflows the viewport; the version sub-caption stays on one line.
3. Hover the Windows button — border and text brighten.
4. Click "See How It Works" — the page scrolls to `#how-it-works`.

- [ ] **Step 7: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "feat(landing): hero windows CTA and edition-neutral badges"
```

---

### Task 3: The `#compare` section

The largest task: comparison table, two guidance panels, and the "install both" callout. Inserted between `#features` and `#pricing`.

**Files:**
- Modify: `index.html` — insert after the `#features` section's closing `</section>` and its following `<div class="section-divider"></div>`, immediately before `<!-- ===== PRICING ===== -->`

**Interfaces:**
- Consumes: the `data-download-edition` / `data-download-location` convention from Task 1; resolves the `#compare` nav links added in Task 1.
- Produces: the `#compare` anchor, referenced later by the FAQ entry in Task 5.

- [ ] **Step 1: Insert the section**

Find this exact sequence (end of the features section, start of pricing):

```html
  <div class="section-divider"></div>

  <!-- ===== PRICING ===== -->
```

Replace it with:

```html
  <div class="section-divider"></div>

  <!-- ===== COMPARE EDITIONS ===== -->
  <section id="compare" class="relative py-24 lg:py-32 bg-base-900">
    <div class="max-w-5xl mx-auto px-6 lg:px-8">
      <div class="text-center max-w-2xl mx-auto mb-14 reveal">
        <div class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-accent-cyan/10 border border-accent-cyan/20 mb-6">
          <span class="font-body text-xs font-semibold text-accent-cyan tracking-wider uppercase">Two Editions</span>
        </div>
        <h2 class="font-display font-extrabold text-3xl sm:text-4xl lg:text-5xl tracking-tight mb-5">
          Extension or <span class="gradient-text">Windows&nbsp;App?</span>
        </h2>
        <p class="font-body text-lg text-slate-400 leading-relaxed">
          Both translate in real time across 60+ languages. They differ in where the sound can come from — and in how much else they do.
        </p>
      </div>

      <!-- Comparison table -->
      <div class="glass-card rounded-3xl overflow-hidden reveal mb-12">
        <div class="overflow-x-auto">
          <table class="w-full min-w-[560px] border-collapse text-left">
            <caption class="sr-only">Feature comparison between the Live Translator browser extension and the Live Translator Windows app</caption>
            <thead>
              <tr class="border-b border-white/10">
                <th scope="col" class="sticky left-0 bg-base-900 z-10 p-5 font-display font-semibold text-sm text-slate-400">Feature</th>
                <th scope="col" class="p-5 font-display font-bold text-sm text-white whitespace-nowrap">
                  <span class="inline-flex items-center gap-2">
                    <svg class="text-accent-blue" viewBox="0 0 24 24" fill="currentColor" style="width:1rem;height:1rem"><path d="M12 0C8.21 0 4.831 1.757 2.632 4.501l3.953 3.953A7.48 7.48 0 0112 4.5c1.77 0 3.397.617 4.682 1.645l3.953-3.953C18.435 1.083 15.382 0 12 0zm0 24c3.79 0 7.169-1.757 9.368-4.501l-3.953-3.953A7.48 7.48 0 0112 19.5a7.48 7.48 0 01-5.415-3.954l-3.953 3.953C5.565 22.243 8.618 24 12 24zM1.757 7.632A11.946 11.946 0 000 12c0 1.58.307 3.09.864 4.472l4.812-2.779A7.444 7.444 0 014.5 12c0-.592.069-1.169.198-1.723L1.757 7.632zm20.486 0l-2.941 2.645c.129.554.198 1.131.198 1.723s-.069 1.169-.198 1.723l2.941 2.645A11.946 11.946 0 0024 12c0-1.58-.307-3.09-.864-4.472z"/></svg>
                    Extension
                  </span>
                </th>
                <th scope="col" class="p-5 font-display font-bold text-sm text-white whitespace-nowrap">
                  <span class="inline-flex items-center gap-2">
                    <svg class="text-accent-cyan" viewBox="0 0 24 24" fill="currentColor" style="width:1rem;height:1rem"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
                    Windows
                  </span>
                </th>
              </tr>
            </thead>
            <tbody class="font-body text-sm">
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Real-time transcription &amp; translation (60+ languages)</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Audio source</th>
                <td class="p-5 text-slate-400">Browser tab + mic</td>
                <td class="p-5 text-white font-medium">Any app on your PC + mic</td>
              </tr>
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Subtitle overlay</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-400"><span class="inline-flex items-center gap-2"><svg class="w-5 h-5 text-accent-emerald flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg>Always-on-top window</span></td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">AI Assistant</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
              </tr>
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Bring your own key (Soniox / OpenAI)</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Translate webpage</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Narrate Their Voice (dubbing)</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Narrate My Voice (two-way translate)</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Screen recording</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Auto-save to Browser / Notion</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Context settings &amp; custom speaker names</th>
                <td class="p-5"><svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg></td>
                <td class="p-5 text-slate-600 text-lg leading-none">—</td>
              </tr>
              <tr class="border-b border-white/5 bg-white/[0.02]">
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Installation required</th>
                <td class="p-5 text-slate-400">No</td>
                <td class="p-5 text-slate-400">Yes — ~96 MB installer</td>
              </tr>
              <tr>
                <th scope="row" class="sticky left-0 bg-base-900 z-10 p-5 font-normal text-slate-300">Platform</th>
                <td class="p-5 text-slate-400">Chrome, Edge, Brave</td>
                <td class="p-5 text-slate-400">Windows 10/11</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Which should I choose? -->
      <div class="text-center max-w-2xl mx-auto mb-10 reveal">
        <h3 class="font-display font-extrabold text-2xl sm:text-3xl tracking-tight mb-3">Which should I choose?</h3>
      </div>

      <div class="grid md:grid-cols-2 gap-6 mb-8">
        <div class="glass-card rounded-3xl p-8 reveal flex flex-col">
          <div class="w-12 h-12 rounded-2xl bg-accent-blue/10 border border-accent-blue/20 flex items-center justify-center mb-5">
            <svg class="w-6 h-6 text-accent-blue" viewBox="0 0 24 24" fill="currentColor"><path d="M12 0C8.21 0 4.831 1.757 2.632 4.501l3.953 3.953A7.48 7.48 0 0112 4.5c1.77 0 3.397.617 4.682 1.645l3.953-3.953C18.435 1.083 15.382 0 12 0zm0 24c3.79 0 7.169-1.757 9.368-4.501l-3.953-3.953A7.48 7.48 0 0112 19.5a7.48 7.48 0 01-5.415-3.954l-3.953 3.953C5.565 22.243 8.618 24 12 24zM1.757 7.632A11.946 11.946 0 000 12c0 1.58.307 3.09.864 4.472l4.812-2.779A7.444 7.444 0 014.5 12c0-.592.069-1.169.198-1.723L1.757 7.632zm20.486 0l-2.941 2.645c.129.554.198 1.131.198 1.723s-.069 1.169-.198 1.723l2.941 2.645A11.946 11.946 0 0024 12c0-1.58-.307-3.09-.864-4.472z"/></svg>
          </div>
          <h4 class="font-display font-bold text-xl mb-3">Choose the Extension if…</h4>
          <p class="font-body text-sm text-slate-400 leading-relaxed mb-4">
            You meet and watch inside your browser and want every feature. This is the most capable version — dubbing, webpage translation, screen recording, auto-save — and there is nothing to install.
          </p>
          <p class="font-body text-sm text-slate-500 leading-relaxed mb-6">
            <strong class="text-slate-400 font-medium">Limitation:</strong> it can only translate audio playing in the browser.
          </p>
          <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" data-download-edition="extension" data-download-location="compare" class="btn-primary relative inline-flex items-center justify-center gap-2 px-6 py-3.5 rounded-2xl font-display font-bold text-sm text-white mt-auto">
            <span class="relative z-10">Add to Browser — It's Free</span>
          </a>
        </div>

        <div class="glass-card rounded-3xl p-8 reveal flex flex-col">
          <div class="w-12 h-12 rounded-2xl bg-accent-cyan/10 border border-accent-cyan/20 flex items-center justify-center mb-5">
            <svg class="w-6 h-6 text-accent-cyan" viewBox="0 0 24 24" fill="currentColor"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
          </div>
          <h4 class="font-display font-bold text-xl mb-3">Choose Windows if…</h4>
          <p class="font-body text-sm text-slate-400 leading-relaxed mb-4">
            You need to translate audio from <strong class="text-white font-medium">any</strong> application — the Zoom or Teams desktop client, a media player, a game, internal software. It captures whatever your PC is playing.
          </p>
          <p class="font-body text-sm text-slate-500 leading-relaxed mb-6">
            <strong class="text-slate-400 font-medium">Trade-off:</strong> fewer features, but no restriction on where the sound comes from.
          </p>
          <a href="./desktop/LiveTranslator-Setup-latest.exe" data-download-edition="windows" data-download-location="compare" class="inline-flex flex-col items-center justify-center px-6 py-3 rounded-2xl border border-white/15 hover:border-white/30 hover:bg-white/5 transition-all duration-300 mt-auto">
            <span class="inline-flex items-center gap-2.5 font-display font-bold text-sm text-slate-200">
              <svg viewBox="0 0 24 24" fill="currentColor" style="width:1rem;height:1rem"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
              Download for Windows
            </span>
            <span class="font-body text-[11px] text-slate-500 mt-0.5">Windows 10/11 · v0.1.4 · ~96 MB</span>
          </a>
          <a href="./instruction.html" class="font-body text-xs text-slate-600 hover:text-slate-400 transition-colors text-center mt-3">Need install help? See the user guide</a>
        </div>
      </div>

      <div class="flex items-start gap-4 p-6 rounded-2xl bg-accent-emerald/5 border border-accent-emerald/15 reveal">
        <div class="w-10 h-10 rounded-xl bg-accent-emerald/10 flex items-center justify-center flex-shrink-0">
          <svg class="w-5 h-5 text-accent-emerald" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg>
        </div>
        <div>
          <div class="font-display font-semibold text-base mb-1">You can install both</div>
          <p class="font-body text-sm text-slate-400 leading-relaxed">
            Running the extension and the Windows app side by side is fully supported. Same account, same license — use whichever fits the moment.
          </p>
        </div>
      </div>
    </div>
  </section>

  <div class="section-divider"></div>

  <!-- ===== PRICING ===== -->
```

- [ ] **Step 2: Verify with grep**

```bash
cd /e/ezitech/live-translator-landing && grep -c 'id="compare"' index.html; grep -c '<tr' index.html; grep -c 'data-download-location="compare"' index.html; grep -c 'Which should I choose' index.html
```

Expected: `1`, `14` (1 header row + 13 feature rows), `2`, `1`.

- [ ] **Step 3: Verify in a browser**

1. Click `Compare` in the nav — the page scrolls to the new section (this resolves the dangling link from Task 1).
2. At 1440 px: the table fits without a horizontal scrollbar; alternating rows are subtly tinted.
3. At 375 px: the table scrolls horizontally *inside its card*; the page body does **not** scroll horizontally (check `document.body.scrollWidth === window.innerWidth` in the console). The "Feature" column stays pinned on the left while scrolling, with a solid background — no text bleeding through.
4. Both guidance cards are equal height and their install buttons align at the bottom (`mt-auto`).
5. The section reveals on scroll like its neighbours.

- [ ] **Step 4: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "feat(landing): add edition comparison section"
```

---

### Task 4: Pricing note and the `#platforms` Windows card

**Files:**
- Modify: `index.html` — pricing footnotes block (~lines 975–982), `#platforms` subtitle (~line 1063) and card grid (~line 1195)

**Interfaces:**
- Consumes: the `data-download-edition` convention from Task 1 (for the pricing-plan CTA already present).
- Produces: nothing later tasks depend on.

- [ ] **Step 1: Add the cross-edition note to pricing**

Find:

```html
      <div class="text-center mt-10 reveal max-w-3xl mx-auto space-y-2">
        <p class="font-body text-sm text-slate-500 leading-relaxed">
          * Unlimited usage with your own API key.
        </p>
```

Replace with:

```html
      <div class="text-center mt-10 reveal max-w-3xl mx-auto space-y-2">
        <p class="font-body text-sm text-slate-300 leading-relaxed">
          Every plan covers <strong class="text-white font-medium">both the browser extension and the Windows app</strong> — one account, one license.
        </p>
        <p class="font-body text-sm text-slate-500 leading-relaxed">
          * Unlimited usage with your own API key.
        </p>
```

- [ ] **Step 2: Add tracking to the Free plan CTA**

Find:

```html
          <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" class="inline-flex items-center justify-center w-full gap-2 px-6 py-3.5 rounded-2xl font-display font-semibold text-sm text-slate-300 border border-white/10 hover:border-white/20 hover:text-white transition-all duration-300 hover:bg-white/5">
            Get Started Free
          </a>
```

Replace with:

```html
          <a href="https://chromewebstore.google.com/detail/live-translator/pohbgdoejhnjneeokhfcfcpodhdllphh" target="_blank" rel="noopener" data-download-edition="extension" data-download-location="pricing" class="inline-flex items-center justify-center w-full gap-2 px-6 py-3.5 rounded-2xl font-display font-semibold text-sm text-slate-300 border border-white/10 hover:border-white/20 hover:text-white transition-all duration-300 hover:bg-white/5">
            Get Started Free
          </a>
```

- [ ] **Step 3: Update the `#platforms` subtitle**

Find:

```html
          Integrates directly into the platforms you use every day for meetings, video content, social media, and online learning.
```

Replace with:

```html
          Integrates directly into the platforms you use every day for meetings, video content, social media, and online learning — and with the Windows app, any other program on your PC.
```

- [ ] **Step 4: Append the "Any Windows App" card**

Find the end of the "Any Website" card and the grid close:

```html
          <h3 class="font-display font-bold text-base mb-2">Any Website</h3>
          <p class="font-body text-xs text-slate-500 leading-relaxed">Turn on <strong class="text-slate-400 font-medium">Show translator on all websites</strong> in Settings to use Live Translator in any website.</p>
        </div>
      </div>
```

Replace with:

```html
          <h3 class="font-display font-bold text-base mb-2">Any Website</h3>
          <p class="font-body text-xs text-slate-500 leading-relaxed">Turn on <strong class="text-slate-400 font-medium">Show translator on all websites</strong> in Settings to use Live Translator in any website.</p>
        </div>

        <!-- Any Windows App -->
        <div class="platform-icon glass-card rounded-2xl p-8 text-center reveal relative">
          <span class="absolute top-3 right-3 inline-flex items-center px-2 py-0.5 rounded-full text-[10px] font-display font-semibold tracking-wide text-accent-cyan bg-accent-cyan/10 border border-accent-cyan/25">Windows app</span>
          <div class="w-16 h-16 mx-auto mb-5 rounded-2xl bg-gradient-to-br from-cyan-500/10 to-blue-500/10 flex items-center justify-center border border-white/5">
            <svg class="w-8 h-8 text-accent-cyan" viewBox="0 0 24 24" fill="currentColor"><path d="M0 3.449L9.75 2.1v9.451H0m10.949-9.602L24 0v11.4H10.949M0 12.6h9.75v9.451L0 20.699M10.949 12.6H24V24l-12.9-1.801"/></svg>
          </div>
          <h3 class="font-display font-bold text-base mb-2">Any Windows App</h3>
          <p class="font-body text-xs text-slate-500 leading-relaxed">Zoom and Teams desktop clients, media players, games, internal tools — anything that plays sound. <a href="#compare" class="text-accent-cyan hover:underline">Compare editions</a>.</p>
        </div>
      </div>
```

- [ ] **Step 5: Verify with grep**

```bash
cd /e/ezitech/live-translator-landing && grep -c 'Any Windows App' index.html; grep -c 'Every plan covers' index.html; grep -c 'data-download-location="pricing"' index.html
```

Expected: `1`, `1`, `1`.

- [ ] **Step 6: Verify in a browser**

1. `#platforms` now shows 13 cards; the `lg:grid-cols-5` grid's last row holds 3 and is not visually broken.
2. The "Compare editions" link inside the new card scrolls to `#compare`.
3. The pricing note renders above the two existing footnotes and is brighter than them.

- [ ] **Step 7: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "feat(landing): windows app in pricing note and platforms grid"
```

---

### Task 5: FAQ entries

**Files:**
- Modify: `index.html` — end of `#faq-container` (after "FAQ 8", ~line 1573)

**Interfaces:**
- Consumes: the `#compare` anchor created in Task 3.
- Produces: nothing later tasks depend on.

- [ ] **Step 1: Append two FAQ items**

Find the end of the last FAQ item and the container close:

```html
              <p class="font-body text-sm text-slate-400 leading-relaxed">
                Absolutely. Your Soniox API key is stored only in your browser's local storage and is never sent to our servers. The extension communicates directly between your browser and Soniox, so you keep full control of key usage and revocation.
              </p>
            </div>
          </div>
        </div>
      </div>
```

Replace with:

```html
              <p class="font-body text-sm text-slate-400 leading-relaxed">
                Absolutely. Your Soniox API key is stored only in your browser's local storage and is never sent to our servers. The extension communicates directly between your browser and Soniox, so you keep full control of key usage and revocation.
              </p>
            </div>
          </div>
        </div>

        <!-- FAQ 9 -->
        <div class="faq-item glass-card rounded-2xl overflow-hidden reveal">
          <button class="w-full flex items-center justify-between p-6 text-left" onclick="toggleFaq(this)">
            <span class="font-display font-semibold text-base pr-4">What is the difference between the extension and the Windows app?</span>
            <span class="faq-icon text-slate-500 flex-shrink-0">
              <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" /></svg>
            </span>
          </button>
          <div class="faq-answer">
            <div class="px-6 pb-6 pt-0">
              <p class="font-body text-sm text-slate-400 leading-relaxed">
                The browser extension has more features — webpage translation, dubbing, screen recording, auto-save — but it can only translate audio playing inside your browser. The Windows app has a smaller feature set, and translates audio from <strong class="text-slate-300 font-medium">any</strong> application on your PC: desktop Zoom or Teams, media players, games, internal tools. See the <a href="#compare" class="text-accent-blue hover:underline">full comparison</a>.
              </p>
            </div>
          </div>
        </div>

        <!-- FAQ 10 -->
        <div class="faq-item glass-card rounded-2xl overflow-hidden reveal">
          <button class="w-full flex items-center justify-between p-6 text-left" onclick="toggleFaq(this)">
            <span class="font-display font-semibold text-base pr-4">Does my Pro license work on both?</span>
            <span class="faq-icon text-slate-500 flex-shrink-0">
              <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" /></svg>
            </span>
          </button>
          <div class="faq-answer">
            <div class="px-6 pb-6 pt-0">
              <p class="font-body text-sm text-slate-400 leading-relaxed">
                Yes. One account and one license cover both editions — there is nothing extra to buy. You can install the extension and the Windows app on the same machine and run them side by side.
              </p>
            </div>
          </div>
        </div>
      </div>
```

- [ ] **Step 2: Verify with grep**

```bash
cd /e/ezitech/live-translator-landing && grep -c 'class="faq-item' index.html; grep -c 'Does my Pro license work on both' index.html
```

Expected: `10`, `1`.

- [ ] **Step 3: Verify in a browser**

1. The FAQ list shows 10 items.
2. Click each new item — it expands, and opening one collapses the others (the existing `toggleFaq` behavior).
3. Click "full comparison" inside FAQ 9 — the page scrolls to `#compare`.

- [ ] **Step 4: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "docs(landing): faq entries for the windows edition"
```

---

### Task 6: Meta tags, Open Graph, and JSON-LD

**Files:**
- Modify: `index.html` — `<head>` (~lines 7, 8, 19, 27) and the `application/ld+json` block (~lines 36–84)

**Interfaces:**
- Consumes: nothing.
- Produces: nothing.

- [ ] **Step 1: Update the meta description**

Find:

```html
  <meta name="description" content="Live Translator is a browser extension for real-time translation in Google Meet, Zoom, Microsoft Teams, YouTube and more. Includes two-way translation, interpreter mode for meeting chat, 60+ languages, optional narration, and private local processing." />
```

Replace with:

```html
  <meta name="description" content="Live Translator is a browser extension and Windows app for real-time translation in Google Meet, Zoom, Microsoft Teams, YouTube and more. The Windows app translates audio from any application on your PC. Two-way translation, interpreter mode, 60+ languages, and private local processing." />
```

- [ ] **Step 2: Update the keywords**

Find:

```html
  <meta name="keywords" content="live translator extension, real-time meeting translation, Google Meet translator extension, Zoom translator extension, Microsoft Teams translator, YouTube video translator, Soniox API translation, OpenAI TTS, narration mode, browser translation extension" />
```

Replace with:

```html
  <meta name="keywords" content="live translator extension, live translator windows app, real-time meeting translation, Google Meet translator extension, Zoom translator extension, Microsoft Teams translator, YouTube video translator, windows translator app, desktop translation app, system audio translation, Soniox API translation, OpenAI TTS, narration mode, browser translation extension" />
```

- [ ] **Step 3: Update the Open Graph description**

Find:

```html
  <meta property="og:description" content="Translate meetings and videos in real time with two-way translation, interpreter mode, and 60+ languages." />
```

Replace with:

```html
  <meta property="og:description" content="Translate meetings and videos in real time — browser extension or Windows app. Two-way translation, interpreter mode, 60+ languages." />
```

- [ ] **Step 4: Update the Twitter description**

Find:

```html
  <meta name="twitter:description" content="Real-time translation with two-way interpreter mode for Google Meet, Zoom, Teams, YouTube and more. Private local processing." />
```

Replace with:

```html
  <meta name="twitter:description" content="Real-time translation for Google Meet, Zoom, Teams, YouTube — plus a Windows app that translates audio from any program on your PC. Private local processing." />
```

- [ ] **Step 5: Update the JSON-LD**

Find:

```json
      "applicationCategory": "BrowserApplication",
      "description": "Live Translator is a browser extension that provides real-time translation for Google Meet, Zoom, Microsoft Teams, and YouTube with private local processing.",
      "operatingSystem": "Chrome, Edge, Brave, Arc",
```

Replace with:

```json
      "applicationCategory": ["BrowserApplication", "DesktopApplication"],
      "description": "Live Translator is a browser extension and Windows app that provides real-time translation for Google Meet, Zoom, Microsoft Teams, and YouTube, with private local processing. The Windows app translates audio from any application on your PC.",
      "operatingSystem": "Chrome, Edge, Brave, Arc, Windows",
```

Then find the last entry of `featureList`:

```json
        "OpenAI Realtime API for speech-to-speech translation"
      ]
```

Replace with:

```json
        "OpenAI Realtime API for speech-to-speech translation",
        "Windows desktop app that translates audio from any application on your PC"
      ]
```

- [ ] **Step 6: Verify with grep and a JSON parse**

```bash
cd /e/ezitech/live-translator-landing && grep -c 'Windows app' index.html; node -e "const h=require('fs').readFileSync('index.html','utf8');const m=h.match(/<script type=\"application\/ld\+json\">([\s\S]*?)<\/script>/);JSON.parse(m[1]);console.log('JSON-LD OK')"
```

Expected: a count of at least `3`, then `JSON-LD OK`.

- [ ] **Step 7: Verify in a browser**

In DevTools console:

```js
document.querySelector('meta[name="description"]').content.includes('Windows app')
document.querySelector('meta[property="og:description"]').content.includes('Windows app')
```

Both return `true`.

- [ ] **Step 8: Commit**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "chore(landing): meta, og and json-ld cover the windows edition"
```

---

### Task 7: Full-page verification pass

No code changes unless a check fails. This is the gate before the page ships.

**Files:**
- Modify: `index.html` only if a check below fails.

**Interfaces:**
- Consumes: everything from Tasks 1–6.
- Produces: nothing.

- [ ] **Step 1: Assert every acceptance criterion by grep**

```bash
cd /e/ezitech/live-translator-landing && \
echo "dead #install anchors (want 0):" $(grep -c 'href="#install"' index.html) && \
echo "windows download links (want 4):" $(grep -c 'desktop/LiveTranslator-Setup-latest.exe' index.html) && \
echo "versioned exe links (want 0):" $(grep -c 'LiveTranslator-Setup-0\.' index.html) && \
echo "download-tracked links (want 9):" $(grep -c 'data-download-edition' index.html) && \
echo "compare section (want 1):" $(grep -c 'id="compare"' index.html) && \
echo "compare anchors (want 4):" $(grep -c 'href="#compare"' index.html) && \
echo "faq items (want 10):" $(grep -c 'class="faq-item' index.html)
```

`grep -c` prints `0` and exits non-zero when there are no matches; inside `$( )` the count still prints, so all seven lines render.

Breakdown behind the expected counts — if a number is off, find the missing
attribute rather than adjusting the expectation:

- **4 Windows download links:** header dropdown, mobile menu, hero, compare card.
- **9 tracked links:** those 4, plus 5 extension links — header dropdown, mobile
  menu, hero, compare card, pricing Free CTA.
- **4 `#compare` anchors:** desktop nav, mobile nav, platforms card, FAQ 9.

- [ ] **Step 2: Check for broken in-page anchors**

```bash
cd /e/ezitech/live-translator-landing && node -e "
const h=require('fs').readFileSync('index.html','utf8');
const ids=new Set([...h.matchAll(/id=\"([^\"]+)\"/g)].map(m=>m[1]));
const refs=[...h.matchAll(/href=\"#([^\"]+)\"/g)].map(m=>m[1]).filter(r=>r);
const bad=[...new Set(refs)].filter(r=>!ids.has(r));
console.log(bad.length? 'BROKEN: '+bad.join(', ') : 'all in-page anchors resolve');
"
```

Expected: `all in-page anchors resolve`.

- [ ] **Step 3: Confirm the download file exists**

```bash
cd /e/ezitech/live-translator-landing && ls -l desktop/LiveTranslator-Setup-latest.exe
```

If this fails, the page is still correct but the download 404s. Flag it to the product owner: the 0.1.4 installer must be uploaded as **both** `LiveTranslator-Setup-0.1.4.exe` (for `latest.yml` auto-update) and `LiveTranslator-Setup-latest.exe` (for the landing page), and `latest.yml` must be regenerated. Do not work around this by linking a versioned filename.

- [ ] **Step 4: Browser pass at 1440 px**

1. Header dropdown opens, closes on outside click and on `Escape`.
2. Nav is one line, 8 links, no wrap.
3. Hero shows two buttons plus the "See How It Works" text link.
4. `#compare` table renders without horizontal scroll; both guidance cards are equal height.
5. Pricing shows the "Every plan covers both" line.
6. `#platforms` shows 13 cards.
7. FAQ shows 10 items; both new ones expand.
8. No errors in the DevTools console.

- [ ] **Step 5: Browser pass at 375 px**

1. `document.body.scrollWidth === window.innerWidth` returns `true` — the page never scrolls horizontally.
2. The comparison table scrolls inside its own card; the "Feature" column stays pinned with an opaque background.
3. Hamburger menu opens and shows `Compare` plus two install buttons; tapping any link closes it.
4. Hero buttons stack without overflowing.

- [ ] **Step 6: Keyboard and analytics pass**

1. Tab from the logo through the nav to "Install Free"; `Enter` opens the panel; `Tab` moves through both entries; `Escape` closes it and returns focus to the button.
2. In the console: `dataLayer.length`, then click each of the 9 tracked links (use middle-click or cancel the navigation), re-checking that the length grows. Confirm one event carries `{edition: 'windows', location: 'hero'}`.

- [ ] **Step 7: Commit any fixes**

```bash
cd /e/ezitech/live-translator-landing && git add index.html && git commit -m "fix(landing): verification pass corrections"
```

If nothing needed fixing, skip this step.

---

## Deliberate omissions

Recorded so a later reader knows these were decisions, not oversights:

- **No test framework.** A static single-file landing page with no build step does not earn Vitest or Playwright. `grep` assertions plus a browser pass cover the same ground at a fraction of the machinery.
- **Version string is hardcoded**, not fetched from `desktop/latest.yml`. A `fetch` plus render logic to display one number is not worth the request; the download link itself never breaks.
- **The 10 `instruction-*.html` pages are untouched.** Windows install instructions are a separate ticket; `#compare` links to the existing `instruction.html` in the meantime.
- **No footer download link.** Explicitly out of scope.
- **Mobile menu has no dropdown** — it is already an expanded disclosure; nesting another one inside adds interaction cost for nothing.
