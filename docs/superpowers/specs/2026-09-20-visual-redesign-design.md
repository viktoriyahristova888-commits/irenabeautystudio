# Irena Beauty Studio — editorial visual redesign

## Context

The site (6 static pages, GitHub Pages, no backend) works and has correct
content, but visually reads as a generic gold-and-black salon template:
italic-gold headings, pill-shaped tabs, four identical square cards per
grid, dark boxed form inputs. The client wants it rebuilt to feel like a
2026 European boutique beauty studio — editorial, photography-led,
near-monochrome, restrained — without losing any working functionality,
content, prices, or contact details, and without inventing content
(no fake stats, testimonials, awards, staff bios).

Full requirements came from the client as an exhaustive creative brief
(colors, typography direction, per-section layout ideas, explicit list of
patterns to avoid, accessibility/performance/SEO targets). This spec
translates that brief into a concrete, buildable system, informed by an
audit of the actual repo and assets.

## Audit findings that shape this spec

- **No mobile navigation exists today.** `.top__nav` and `.top__phone` are
  simply `display:none` under 760px with no replacement. This is a
  functional gap, not just a style problem — the new mobile menu is a
  required build, not a restyle.
- `pricing.html` already uses a `:has()`-driven CSS tab system (no JS) with
  hairline dividers — structurally close to the "menu" concept the brief
  wants. Treat as reskin, not rebuild.
- `extensions.html` and `gallery.html` already have decent bones (lazy
  video swatches, FAQ accordion, asymmetric collage grid, a working
  `<dialog>` lightbox with keyboard nav). Reskin + targeted structural
  upgrades, not rewrites.
- `index.html` and `services.html` both reuse a `.funnel-grid` of four
  equal `aspect-ratio:3/4` cards — exactly the repetitive card-grid
  pattern the brief wants replaced with asymmetric editorial mosaics.
- Real usable before/after pair found: `images/donika-before.webp` (wet,
  untreated hair) and `images/r-donika1.webp` (same chair, same wall
  clock, same shoulder — finished result). Confirmed via visual
  inspection these are the same session. Use for the interactive
  before/after slider instead of the current arbitrary two-photo grid.
- **`обработени снимки/салон/nobaner.jpeg` shows shelving branded "IBS
  Beauty Studio"** — a different business, not Irena's. Exclude from all
  use. Flagged to the client; not this project's file to delete.
- Video assets referenced in `extensions.html` (`videos/process/*`,
  `videos/swatch/*`) all exist on disk and work — keep this feature,
  reskin only.
- Fonts currently loaded: Playfair Display + Jost, both already verified
  rendering Bulgarian correctly live (checked earlier this project).

## Font decision (verified, not assumed)

Checked actual Google Fonts `css2` responses (with a browser user-agent,
since the default UA collapses to a single latin-only block) for
`unicode-range` coverage of `U+0400-045F` (Cyrillic):

- **Bodoni Moda** — brief's suggested "editorial Bodoni" reference —
  **has no Cyrillic block at all.** Rejected; would silently fall back to
  a system font for every Bulgarian character.
- **Manrope** — confirmed `unicode-range: U+0400-045F` block present.
  Adopted as the interface/body sans, replacing Jost.
- **Playfair Display** stays as the sole display serif (already
  integrated, already Cyrillic-verified live in this project) but its
  *usage* changes: fewer places, larger sizes, no gold-italic-everywhere,
  reserved for oversized editorial statements and numerals.

Google Fonts link (replaces the current one in every page `<head>`):
```html
<link href="https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700&family=Playfair+Display:ital,wght@0,500;0,600;1,500&display=swap" rel="stylesheet">
```

## Design tokens (`styles.css` `:root`, full replacement)

```css
:root{
  --ink:#111111;
  --ink-soft:#1b1b1b;
  --ivory:#f6f4f0;
  --white:#ffffff;
  --stone:#c8c0b7;
  --taupe:#a99c8d;      /* sparing accent only — never a fill color */
  --line:rgba(17,17,17,.12);        /* on light backgrounds */
  --line-dark:rgba(246,244,240,.14); /* on dark backgrounds */
  --serif:'Playfair Display',Georgia,serif;
  --sans:'Manrope',Arial,sans-serif;
  --radius:3px;
  --container:1440px;
}
```

Body copy sits on `--ivory`/`--white` by default (a lighter, airier field
than the current all-black site — closer to the editorial-magazine
reference than a nightclub-dark salon). Dark sections (`--ink`) are used
deliberately for contrast — hero overlays, footer, the alternating
"technology/credibility" band — not as the default canvas.

Buttons: solid `--ink` primary with `--ivory` text, `--radius` (not a
pill); secondary is text + arrow with an underline/border that animates
on hover (slide 4px), no fill. No corner radius above 4px anywhere on the
site.

## Shared components (built once in Phase 1, reused everywhere)

**Header** — logo wordmark left (refined type treatment, not the current
serif-italic-em logo), nav center-right in tracked-out uppercase Manrope,
"ЗАПАЗИ ЧАС" as a bordered/solid button, phone number small and
secondary. Transparent over a hero; adds a solid `--ivory`/`--ink`
background + border-bottom via a scroll-triggered class (`IntersectionObserver`
on a 1px sentinel at top of page, consistent with the existing reveal-on-scroll
pattern already in every page's script block).

**Mobile navigation (new — currently does not exist)** — a hamburger
icon (two lines, no circle/box around it) opens a full-screen `--ink`
overlay: large stacked Manrope links, the booking CTA, then phone/Viber/
Facebook as a row at the bottom. Closes on link click, on backdrop
click if any, and on `Escape`. Built with a native `<dialog>` or a
simple `hidden`-toggle + `aria-expanded`, matching the accessibility
pattern already used by the gallery lightbox (`<dialog>`, focus handled
by the browser).

**Persistent mobile booking bar** — fixed bottom strip, safe-area
padding (`env(safe-area-inset-bottom)`), shown only below 760px, hidden
via `IntersectionObserver` once the visitor scrolls to `#book` on
`contact.html` (so it never overlaps the real form), not shown at all
if that would duplicate a CTA already on-screen.

**Footer** — dark, address/hours/phone/Viber/Facebook/Google column,
nav column, CTA, oversized "IRENA BEAUTY STUDIO" typographic wordmark
at the bottom as a graphic element (large Playfair Display, low-opacity
or outline treatment, not competing with real content above it).

**Before/after slider** — a single reusable component: two stacked
images (`donika-before.webp` under, `r-donika1.webp` on top, clipped via
`clip-path`/`width` on a wrapper), a draggable handle controlled by
pointer events (`pointerdown/move/up`, works for mouse and touch without
separate code paths), `ПРЕДИ`/`СЛЕД` labels, keyboard-operable (arrow
keys move the handle when focused, per the accessibility requirement).
Falls back to a static side-by-side pair if JS fails to init.

## Phase 1 scope (this implementation pass)

1. Rewrite `styles.css` tokens/base/header/footer/hero/button/mobile-nav
   rules described above. Remove now-dead rules only after confirming no
   page still references them (cross-check against services/extensions/
   pricing/gallery/contact inline `<style>` blocks before deleting
   anything shared).
2. Rebuild `index.html`: cinematic hero (existing `hero-main.webp`,
   ~90vh, gradient only where the text sits, not across the whole
   frame), brand-statement band using existing positioning copy (no new
   claims), asymmetric mosaic replacing `.funnel-grid` (large/medium/
   narrow/small tiles, staggered vertical offsets — reuse the stagger
   technique already used in `.swatch-strip` on `extensions.html`),
   shared footer.
3. Build header/mobile-nav/footer as shared markup blocks, duplicated
   consistently across pages (this codebase has no templating layer —
   confirmed no build step exists — so each HTML file keeps its own
   copy of the header/footer markup, same as today).
4. Verify: local Playwright pass on `index.html` at 1280px and a real
   mobile viewport (390px, `is_mobile`, `has_touch`) — zero console
   errors, zero horizontal overflow, mobile menu opens/closes/traps
   focus reasonably, hero renders correctly, `prefers-reduced-motion`
   respected (existing `reveal` class pattern already honors it — keep
   that mechanism).
5. Show the client the live result before starting Phase 2.

## Phase 2 scope (next pass, after Phase 1 sign-off, same spec — no new brainstorming needed)

Applies the now-proven system to the remaining five pages, per the
per-page notes already agreed in this spec:

- `services.html` — same asymmetric mosaic pattern as the homepage,
  four services, fully-clickable cards kept.
- `extensions.html` — promoted to a signature-service treatment; keep
  the process-video strip and swatch-video strip (reskin only); replace
  the static before/after image pair with the new slider component
  using the confirmed `donika` pair; reskin the FAQ accordion and the
  products/technology credibility section.
- `pricing.html` — reskin only: remove pill-shaped tabs in favor of a
  minimal underline/text tab row, clean hairline-divider rhythm, keep
  the existing `:has()`-driven interactivity as-is (it already works and
  needs no JS).
- `gallery.html` — reskin; replace the in-browse `<select>` category
  dropdown with the same minimal text-link filter style used elsewhere,
  for consistency with the entry picker; keep the asymmetric collage
  grid and the existing lightbox behavior.
- `contact.html` — restructure to a desktop split-screen (large image
  left using `hero-contact.webp`, facts + form right), stacking to a
  single column (image first, cropped shorter) on mobile; reskin form
  inputs to minimal bordered/underlined fields with clear focus states;
  keep the FormSubmit integration and the honeypot field exactly as-is.

## Content rules (unchanged from the client's brief)

Bulgarian stays the language of the site. Headings/CTA microcopy may be
tightened for tone, but: no changed prices, no changed phone/address, no
invented services, no invented reviews/testimonials/stats/staff bios/
awards. Where the brief listed cliché phrases to avoid ("Открийте
красотата във вас" etc.), do not introduce equivalents.

## Non-goals

- No framework migration (stays static HTML/CSS/vanilla JS — no genuine
  reason to change architecture, per the client's own brief).
- No new backend, no new third-party services beyond what's already
  integrated (FormSubmit, Google Fonts, Google Maps embed).
- No fabricated imagery — only existing repo assets (`images/`,
  `videos/`) plus the two confirmed-real `донika` before/after photos.
  The mismatched-brand photo is excluded, not replaced with a stock
  substitute.

## Verification plan (both phases)

Same method already established and proven this session: local
Playwright checks (`channel='chrome'`) at 1280×900 and a real mobile
emulation (390×844, `device_scale_factor=2`, `is_mobile=True`,
`has_touch=True`) — console-error listener, `requestfailed` listener,
horizontal-overflow check (`scrollWidth - clientWidth === 0`). Widen to
the client's explicit breakpoint list (320/360/375/390/430/768/1024/1440)
for the final Phase 2 sign-off sweep specifically, since that pass is
the one the client will actually judge the finished site by. After each
phase: commit, push, poll GitHub Pages build status until `built`, then
re-run the same checks against the *live* URL before calling it done.
