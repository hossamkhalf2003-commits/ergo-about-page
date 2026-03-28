# Ergo landing page — documentation

This document describes the single-page site implemented in [`latest.html`](latest.html): structure, behavior, and how to customize it.

## Overview

- **What it is:** A static marketing / “about” landing page for Ergo (workforce outsourcing, Egypt), with bilingual UI (English / Arabic), light/dark theme, and forms that POST to Google Apps Script web apps.
- **Delivery model:** One HTML file with embedded CSS and JavaScript—no build step. Fonts load from Google Fonts; hero image from Pexels; client logos are expected under a `logos/` folder relative to the page.

## File layout

| Path | Role |
|------|------|
| `latest.html` | Full page: markup, styles, scripts |
| `logos/*` | Client logo images referenced by `appData.clients` (e.g. `logos/pepsi-logo.png`) |

If you open `latest.html` without a local server, some browsers restrict `file://` requests; use a simple HTTP server when testing logos and forms.

## Tech stack

- **HTML5** — semantic sections, forms, modals with basic ARIA (`role="dialog"`, `aria-modal`, `aria-live` on form status regions).
- **CSS** — custom properties (`:root` / `body.dark-mode`), Flexbox and Grid, `@media` for ≤768px and ≤600px, `backdrop-filter` on the navbar (graceful degradation where unsupported).
- **JavaScript (vanilla)** — no frameworks; `DOMContentLoaded` bootstraps language, theme, observers, and form handlers.

## Page sections (anchor IDs)

| ID | Content |
|----|---------|
| *(top)* | Fixed navbar + hero |
| `#about` | About lead + three “pillar” value cards (trust/compliance/scale) |
| `#services` | Service cards (generated) |
| `#available-jobs` | Position cards (generated) |
| `#stats` | Intro line + animated counters (generated) |
| `#jobs` | Blue / white collar categories + modals |
| `#leaders` | Leader cards (generated) |
| `#clients` | Client logos + “factories” modals |
| `#location` | Address copy + embedded Google Map |
| `#cta-partner` | Conversion banner (gradient CTA to `#contact`; **not** part of nav scroll-spy) |
| `#contact` | Contact form |

Footer holds email, WhatsApp, and copyright line.

## User-facing features

- **Language:** EN / AR toggles `document.documentElement.lang`, `dir` (`ltr` / `rtl`), and replaces all visible strings that expose a `data-key` matching keys in `appData.translations`.
- **Theme:** Toggles `body` class `dark-mode`; persisted as `localStorage.theme` (`dark-mode` or `light`). On load, only `dark-mode` is re-applied (no stray `light` class).
- **Navbar:** Glass-style bar, compact “scrolled” state after ~50px scroll; mobile hamburger opens dropdown; `aria-expanded` / `aria-label` update with menu state.
- **Scroll:** Smooth scroll to anchors; “back to top” button after 100px; **IntersectionObserver** adds `is-visible` to sections for fade/slide-in and underline animation; second observer highlights the active nav link (`active-link`) using `rootMargin: -30% 0 -70% 0` (sections with `id="cta-partner"` are **excluded** so the highlight does not clear while that band is on screen).
- **Stats:** When `#stats` enters view, numbers animate toward `data-target`, then show `data-text` (e.g. `14+`, `+3000`).
- **Modals:** Service details, client location lists, job category lists, job application form—opened via delegated clicks on `body`; closing via overlay (`.modal`) or `.close-btn`.

## Data model (`appData`)

All dynamic lists and copy live in one `appData` object inside the script.

### `translations`

- Keys `en` and `ar` are flat objects: `someKey: "string"`.
- Any element with `data-key="someKey"` gets `textContent` set from the active language when `setLanguage` runs (including after initial load).

**Conventions used elsewhere:**

- Services: `service1Title`, `service1Text`, `service1ModalTitle`, `service1ModalContent`, … through `service6`.
- Jobs grid: `jobTitleLabor`, `jobDescLabor`, … (title vs `Desc` pairs).
- Leaders: `leader1Name`, `leader1Role`, `leader1Bio`, … for `leader1`–`leader3`.
- Forms and UI: `formCompany`, `formBtn`, `jobOptLabor`, `footerText`, etc.
- **Hero & conversion:** `heroBadge`, `heroTitle`, `heroSubtitle`, `heroTrust`, `heroBtnPrimary` (→ `#contact`), `heroBtn` (→ `#services`), `heroBtnApp`; **About pillars:** `pillar1Title`–`pillar3Text`; **Section subtitles:** `aboutSubtitle`, `servicesSubtitle`, `availableJobsSubtitle`, `statsIntro`, `jobsSubtitle`, `leadersSubtitle`, `clientsSubtitle`, `contactLead`; **CTA band:** `ctaBannerTitle`, `ctaBannerText`, `ctaBannerBtn`.

Add or change copy by editing both languages for the same keys.

### `services`

Array of `{ key: 'service1' … 'service6', icon: '<svg>…</svg>' }`. Icons are built with a small `svg(inner)` helper. Card titles/bodies come from translations (`serviceNTitle`, `serviceNText`).

### `positions`

Array of `{ key: 'jobTitleLabor', …, icon }` aligned with translation keys for titles and matching `jobDesc*` keys.

### `stats`

Array of `{ target, text, key, icon }`. `target` is the numeric end value for the animation; `text` is the final label shown (can include `+`). `key` points to a translation for the caption (e.g. `stat1Desc`).

### `leaders`

Array of `{ key: 'leader1' | 'leader2' | 'leader3' }`; names/roles/bios come from translations.

### `clients`

Each entry: `{ key, logo, locations: { en: [...], ar: [...] } }`.  
`logo` is a path relative to the HTML file (e.g. `logos/pepsi-logo.png`). The modal title uses `key` (capitalized) plus the translated `factoriesBtn` string.

### `jobListings`

- `blue` and `white` each have `en` and `ar` with `title` and `jobs` (string arrays).
- Used when the user clicks “See Details” on the job category cards (`data-job-category`).

## Forms and Google Apps Script

Two forms POST with `fetch` + `FormData`:

1. **Contact** — `#contactForm`, hidden field `formName=contact`, status `#contactFormStatus`.
2. **Job application** — `#jobAppForm`, hidden field `formName=jobApplication`, status `#jobAppFormStatus`.

Script URLs are set in JavaScript:

```text
contactScriptURL
jobAppScriptURL
```

Currently both point to the same deployment URL; the backend should branch on `formName` (or you can deploy **two separate** web apps and assign different URLs).

The script is expected to respond with JSON: success when `result === 'success'`, otherwise an error object (e.g. `{ error: '…' }`). The UI shows a short success/error message and resets the button label after a timeout (EN/AR labels for submit buttons are hardcoded in `wireForm`’s `errBtn` object).

**Security note:** Published script URLs are visible in the page source. Restrict usage in Apps Script (e.g. validation, quotas, optional auth) as needed.

## Customization checklist

1. **SEO / sharing** — `<title>`, `<meta name="description">`, keywords, `og:*` tags, canonical URL if you add one.
2. **Branding** — Favicon (data URL in `<link rel="icon">`), hero `background` URL in CSS, `--pab` / `--pdb` (accent and primary blue) in `:root` and `body.dark-mode`.
3. **Contact details** — Footer `mailto:` / `href` for WhatsApp; map `iframe` `src` under `#location`.
4. **Copy** — Prefer editing `translations` keys so EN/AR stay aligned; keep `data-key` on HTML in sync with new keys.
5. **Clients** — Add rows to `appData.clients` and place image files under `logos/`.
6. **Forms** — Replace `contactScriptURL` / `jobAppScriptURL` with your deployed web app URLs.

## Local storage

| Key | Values | Purpose |
|-----|--------|---------|
| `language` | `en`, `ar` | Remember language |
| `theme` | `dark-mode`, `light` | Remember theme preference |

Initial language: saved value, else browser language `ar` → Arabic, otherwise English.

## Dependencies (external)

- `fonts.googleapis.com` / `fonts.gstatic.com` — Cairo + Poppins.
- Hero image — `images.pexels.com` (URL in CSS).
- Optional — Google Maps embed in `#location`.

## Accessibility and UX notes

- Focus outline uses the accent color (`--pab`).
- Form controls use labels; job modal fields include `autocomplete` where relevant.
- Modals use `role="dialog"` and close controls with `aria-label="Close"`.
- For production, consider trapping focus inside open modals and returning focus to the trigger on close (not implemented in the current script).

## Maintenance tips

- After editing minified one-line objects, validate the file (e.g. run the embedded script through a JS parser) to catch missing commas or quotes.
- When adding a new `data-key`, add it to **both** `en` and `ar` in `translations`.
- Re-test RTL: layout uses `html[dir=rtl]` for hero alignment, modal close button, and scroll-to-top button position.

---

*Last aligned with `latest.html` structure and behavior as of project documentation creation.*
