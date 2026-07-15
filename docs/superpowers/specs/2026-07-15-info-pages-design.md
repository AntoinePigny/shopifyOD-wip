# ODEVIE information pages — design

**Date:** 2026-07-15
**Branch:** `feature/info-pages`
**Status:** approved by user (brainstorming session 2026-07-15)

## Goal

Three information pages, one per card in the homepage feature-grid section. Each page shows the theme header, the page title, three drop-downs (title + a few paragraphs + 2–3 images each), and the theme footer. All drop-down content — titles, paragraphs, images — is editable per page in the Shopify theme editor. Layout code exists once and is reused by all three pages.

## Key constraint

Shopify stores theme-editor content per *template file*, not per page. One template assigned to three pages would show identical content everywhere. Distinct content therefore requires one thin template file per page, all sharing a single custom section that holds the layout code.

## Decisions made during brainstorming

- **Content:** distinct per page, edited in the theme editor.
- **Design source:** no mockup; match the existing ODEVIE homepage style (Gantari body, Roustel display, existing color schemes).
- **Drop-down behavior:** independent open/close, several may be open at once, all closed on page load. No JavaScript on the storefront — native `<details>`/`<summary>`; a small design-mode-only script (theme editor exclusively) auto-opens a drop-down when the merchant selects it or one of its blocks, and after editor re-renders.
- **Row layout inside an open drop-down:** alternating text/image rows (layout B): odd rows image-right, even rows image-left on desktop; stacked text-then-image on mobile.
- **Page top:** the page title (from the Shopify page object) renders above the drop-downs via the stock `main-page` section. Page body stays empty in admin.

## Files

### New: `sections/info-accordion.liquid`

One section instance = one drop-down. Loads `odevie-theme.css` like the other ODEVIE sections.

**Section settings**

| id | type | default | notes |
|----|------|---------|-------|
| `title` | text | `Titre de la rubrique` | drop-down heading shown in the summary bar |
| `color_scheme` | color_scheme | `scheme-1` | consistent with other ODEVIE sections |
| `padding_top` | range 0–100 step 4 | 8 | low default so consecutive drop-downs stack tightly |
| `padding_bottom` | range 0–100 step 4 | 8 | |

**Blocks** — single type `row`, max 6:

| id | type | notes |
|----|------|-------|
| `text` | richtext | one or a few paragraphs; defaults to a short French placeholder paragraph (`<p>Ajoutez votre texte ici.</p>`) so new rows render visibly |
| `image` | image_picker | optional, no default |

**Preset:** "Info drop-down" with 2 `row` blocks (default placeholder text, no images) so "Add section" starts usable.

**Markup & behavior**

- Native `<details>`/`<summary>`; closed by default; no JS. Summary bar shows the title plus Dawn's existing `icon-caret` snippet, rotated via CSS when open.
- Blank title renders `&nbsp;` in the summary bar so the drop-down stays visible and fixable in the theme editor.
- Row rendering: text and image side by side on desktop; alternation done purely in CSS with `:nth-child(even) { flex-direction: row-reverse; }`. Row with no image → full-width text. Row with only an image → full-width image, capped height. Row with neither → skipped.
- Mobile (<750px): each row stacks, text first then image.
- Images via `image_url | image_tag` with responsive widths and `loading="lazy"`; alt falls back from image alt to the drop-down title.
- `{{ block.shopify_attributes }}` on each row wrapper for theme-editor deep linking.

**Styling** — appended to `assets/odevie-theme.css`, scoped under `.odevie-info-accordion`, following the file's existing organization (`odevie-feature-grid` etc.): Roustel for the summary title, Gantari body text, hairline divider between stacked drop-downs, existing color-scheme variables throughout.

### New: three template files

`templates/page.info-mineraux.json`, `templates/page.info-richesse.json`, `templates/page.info-rituel.json`.

Each contains, in order:

1. `main-page` (stock Dawn — renders the page title as `h1`; empty page body renders nothing else)
2. three `info-accordion` sections pre-filled with French placeholder titles and 2 rows each (default placeholder text, no images), so every page renders sensibly before real content is entered

Merchants can reorder, add a 4th drop-down, or remove one per page in the theme editor without code changes.

### Modified: `templates/index.json`

The three feature-grid card blocks get links, in the preset's existing card order:

| card | link |
|------|------|
| Pourquoi sommes-nous si nombreux à manquer de minéraux ? | `/pages/mineraux` |
| La richesse naturelle de l'eau de mer, prouvée et préservée | `/pages/richesse` |
| Un rituel simple dans votre quotidien | `/pages/rituel` |

`sections/feature-grid.liquid` itself is unchanged — it already renders card links.

### Modified: `.gitignore`

Add `.superpowers/` (brainstorming companion artifacts).

## One-time admin setup (manual, documented not automated)

In the Shopify dashboard: create pages with handles `mineraux`, `richesse`, `rituel`; leave the body empty; assign each its `page.info-*` template. Until this is done the feature-grid links 404 — acceptable during development, prerequisite for going live.

## Edge cases

- Page missing in admin → link 404s (see admin setup above).
- Blank drop-down title → summary bar still renders (non-breaking space).
- Empty rows / image-only rows / text-only rows → handled per row rendering rules above.
- No JS → no script failure modes; `<details>` is supported everywhere the theme already runs.

## Verification

Via the project's `/verify` flow:

- `theme check` passes (Liquid + schema lint).
- Local dev preview: drop-downs open/close independently, start closed; alternating rows on desktop; stacking on mobile; images lazy-load with alt text.
- Rendered-HTML checks: accordion markup present, feature-grid card `href`s point to the three page URLs.
- Theme editor loads the three templates without schema errors; editing a title/row/image on one page does not affect the others.

## Out of scope

- Creating the store pages via API/automation.
- Changes to `sections/feature-grid.liquid`, header, or footer.
- Exclusive accordion behavior (one-open-at-a-time) — explicitly not wanted.
- The pending homepage mockup assets (hero, engagement, footer coral, wordmark) — separate effort.
