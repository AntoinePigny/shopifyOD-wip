# ODEVIE Info Pages Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Three theme-editor-editable information pages (drop-downs with alternating paragraph/image rows), linked from the homepage feature-grid cards.

**Architecture:** One new Liquid section (`info-accordion`) where one section instance = one drop-down, holding `row` blocks (paragraph + optional image). Three thin JSON page templates each stack `main-page` (page title) + three `info-accordion` instances. `templates/index.json` points the three feature-grid cards at the three pages.

**Tech Stack:** Shopify Liquid (Dawn-based theme), JSON templates, plain CSS appended to `assets/odevie-theme.css`. No JavaScript.

**Spec:** `docs/superpowers/specs/2026-07-15-info-pages-design.md` (approved 2026-07-15).

## Global Constraints

- Branch: `feature/info-pages` (already created and checked out).
- No JavaScript — drop-downs are native `<details>`/`<summary>`, all closed on load, independent.
- Page handles are exactly `mineraux`, `richesse`, `rituel`; template files are exactly `page.info-mineraux.json`, `page.info-richesse.json`, `page.info-rituel.json`.
- French placeholder copy, verbatim from spec: drop-down title default `Titre de la rubrique`; row text default `<p>Ajoutez votre texte ici.</p>`.
- Section defaults: `color_scheme` = `scheme-1`, `padding_top`/`padding_bottom` = 8 (range 0–100 step 4).
- Row blocks: max 6 per drop-down.
- Follow existing ODEVIE conventions: classes prefixed `odevie-info-accordion__`, CSS appended to `assets/odevie-theme.css`, breakpoint 750px, colors via `--color-*` variables. The `.script-accent` class (Roustel, coral) already exists — reuse it for drop-down titles.
- `sections/feature-grid.liquid`, header, and footer are NOT modified.
- Theme check has 2 known pre-existing errors (ValidSchemaTranslations in `sections/main-product.liquid`) plus ~9 warnings. These are not regressions — do not fix them, and do not add any new offense.
- Every commit message ends with the trailer: `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`

**Testing note:** This is a Shopify theme — there is no unit-test framework. The test cycle per task is the project's `/verify` flow (`.claude/skills/verify/SKILL.md`): `shopify theme check` for lint, plus `curl` checks against the rendered storefront served by `shopify theme dev` (auth cached for test-odevie.myshopify.com; the port is dynamic — read it from the log). "Expected:" lines below are the pass criteria; treat a mismatch like a failing test.

---

### Task 1: `info-accordion` section + CSS

**Files:**
- Create: `sections/info-accordion.liquid`
- Modify: `assets/odevie-theme.css` (append at end of file)

**Interfaces:**
- Consumes: `.script-accent` and `--color-foreground` from `assets/odevie-theme.css`/`base.css`; Dawn's `summary`/`summary .icon-caret` base styles (`assets/base.css:651-666`); `assets/icon-caret.svg` (root element `<svg class="icon icon-caret" …>`).
- Produces: section type `info-accordion` with settings `title` (text), `color_scheme`, `padding_top`, `padding_bottom`; block type `row` with settings `text` (richtext), `image` (image_picker). Task 2's templates reference these exact ids.

- [ ] **Step 1: Create `sections/info-accordion.liquid`**

```liquid
{{ 'odevie-theme.css' | asset_url | stylesheet_tag }}

{%- style -%}
  .section-{{ section.id }}-padding {
    padding-top: {{ section.settings.padding_top }}px;
    padding-bottom: {{ section.settings.padding_bottom }}px;
  }
{%- endstyle -%}

<div class="color-{{ section.settings.color_scheme }} gradient section-{{ section.id }}-padding">
  <div class="page-width odevie-info-accordion">
    <details id="Details-{{ section.id }}" class="odevie-info-accordion__details">
      <summary id="Summary-{{ section.id }}" class="odevie-info-accordion__summary">
        <h2 class="odevie-info-accordion__title script-accent">
          {%- if section.settings.title != blank -%}
            {{ section.settings.title | escape }}
          {%- else -%}
            &nbsp;
          {%- endif -%}
        </h2>
        {{- 'icon-caret.svg' | inline_asset_content -}}
      </summary>
      <div
        class="odevie-info-accordion__content"
        role="region"
        aria-labelledby="Summary-{{ section.id }}"
      >
        {%- for block in section.blocks -%}
          {%- if block.settings.text == blank and block.settings.image == blank -%}
            {%- continue -%}
          {%- endif -%}
          <div
            class="odevie-info-accordion__row{% if block.settings.image == blank %} odevie-info-accordion__row--text-only{% endif %}{% if block.settings.text == blank %} odevie-info-accordion__row--image-only{% endif %}"
            {{ block.shopify_attributes }}
          >
            {%- if block.settings.text != blank -%}
              <div class="odevie-info-accordion__text rte">{{ block.settings.text }}</div>
            {%- endif -%}
            {%- if block.settings.image != blank -%}
              {%- assign image_alt = block.settings.image.alt | default: section.settings.title -%}
              <div class="odevie-info-accordion__image">
                {{
                  block.settings.image
                  | image_url: width: 1500
                  | image_tag:
                    loading: 'lazy',
                    widths: '375, 550, 750, 1100, 1500',
                    sizes: '(min-width: 750px) 40vw, calc(100vw - 3rem)',
                    alt: image_alt
                }}
              </div>
            {%- endif -%}
          </div>
        {%- endfor -%}
      </div>
    </details>
  </div>
</div>

{% schema %}
{
  "name": "Info drop-down",
  "tag": "section",
  "class": "section",
  "disabled_on": {
    "groups": ["header", "footer"]
  },
  "max_blocks": 6,
  "settings": [
    { "type": "text", "id": "title", "label": "Drop-down title", "default": "Titre de la rubrique" },
    { "type": "color_scheme", "id": "color_scheme", "label": "Color scheme", "default": "scheme-1" },
    { "type": "range", "id": "padding_top", "min": 0, "max": 100, "step": 4, "unit": "px", "label": "Padding top", "default": 8 },
    { "type": "range", "id": "padding_bottom", "min": 0, "max": 100, "step": 4, "unit": "px", "label": "Padding bottom", "default": 8 }
  ],
  "blocks": [
    {
      "type": "row",
      "name": "Row",
      "settings": [
        { "type": "richtext", "id": "text", "label": "Paragraph", "default": "<p>Ajoutez votre texte ici.</p>" },
        { "type": "image_picker", "id": "image", "label": "Image" }
      ]
    }
  ],
  "presets": [
    {
      "name": "Info drop-down",
      "blocks": [{ "type": "row" }, { "type": "row" }]
    }
  ]
}
{% endschema %}
```

Notes for the implementer:
- `inline_asset_content` inlines the caret SVG exactly like `sections/collapsible-content.liquid:90` does.
- Dawn's `base.css` already gives `summary` `cursor: pointer; list-style: none; position: relative` and absolutely positions `.icon-caret` at `right: 1.5rem` — the CSS in Step 2 builds on that, don't duplicate it.
- A block whose text AND image are both blank is skipped (`{%- continue -%}`); blank-title sections render `&nbsp;` in the summary so they stay clickable in the theme editor.

- [ ] **Step 2: Append accordion CSS to `assets/odevie-theme.css`**

Append at the end of the file (after the last existing rule):

```css

/* Info accordion (info pages: one section instance = one drop-down).
   base.css already styles summary + absolutely positions .icon-caret. */
.odevie-info-accordion__details {
  border-bottom: 0.1rem solid rgba(var(--color-foreground), 0.15);
}

.odevie-info-accordion__summary {
  padding: 1.6rem 3.5rem 1.6rem 0; /* right padding clears the caret from base.css */
}

.odevie-info-accordion__summary .icon-caret {
  transition: transform 0.2s ease;
}

.odevie-info-accordion__details[open] > .odevie-info-accordion__summary .icon-caret {
  transform: rotate(180deg);
}

.odevie-info-accordion__title {
  margin: 0;
  font-size: 2.4rem;
  line-height: 1.2;
}

.odevie-info-accordion__content {
  display: flex;
  flex-direction: column;
  gap: 2.4rem;
  padding: 0.8rem 0 2.4rem;
}

.odevie-info-accordion__row {
  display: flex;
  flex-direction: column;
  gap: 1.6rem;
}

.odevie-info-accordion__image img {
  display: block;
  width: 100%;
  height: auto;
  border-radius: 1.2rem;
}

.odevie-info-accordion__row--image-only .odevie-info-accordion__image img {
  max-height: 42rem;
  object-fit: cover;
}

@media screen and (min-width: 750px) {
  .odevie-info-accordion__title {
    font-size: 3rem;
  }

  .odevie-info-accordion__row {
    flex-direction: row;
    align-items: center;
    gap: 3.2rem;
  }

  /* Layout B: alternate image side by rendered row position
     (all children of __content are rows, so :nth-child parity = row parity) */
  .odevie-info-accordion__row:nth-child(even) {
    flex-direction: row-reverse;
  }

  .odevie-info-accordion__text {
    flex: 1 1 60%;
    min-width: 0;
  }

  .odevie-info-accordion__image {
    flex: 1 1 40%;
    min-width: 0;
  }
}
```

Below 750px every row stacks text-then-image (source order), which is the spec's mobile behavior.

- [ ] **Step 3: Run theme check**

```bash
cd /home/pwaps/odevie && shopify theme check --fail-level error 2>&1 | tail -25
```

Expected: the only errors are the 2 known pre-existing `ValidSchemaTranslations` offenses in `sections/main-product.liquid`. **Zero offenses mention `info-accordion.liquid` or `odevie-theme.css`.** If any do, fix them before proceeding.

- [ ] **Step 4: Commit**

```bash
cd /home/pwaps/odevie && git add sections/info-accordion.liquid assets/odevie-theme.css && git commit -m "Add info-accordion section: one drop-down per instance, alternating rows

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: Three `page.info-*` templates

**Files:**
- Create: `templates/page.info-mineraux.json`
- Create: `templates/page.info-richesse.json`
- Create: `templates/page.info-rituel.json`

**Interfaces:**
- Consumes: section type `info-accordion` (Task 1) with setting ids `title`, `color_scheme`, `padding_top`, `padding_bottom` and block type `row` with setting id `text`; stock section type `main-page`.
- Produces: template suffixes `info-mineraux`, `info-richesse`, `info-rituel` — used by Task 2's `?view=` render check and by the one-time admin page assignment (see Manual follow-up).

- [ ] **Step 1: Create `templates/page.info-mineraux.json`**

```json
{
  "sections": {
    "main": {
      "type": "main-page",
      "settings": {
        "padding_top": 28,
        "padding_bottom": 16
      }
    },
    "accordion-1": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Le rôle des minéraux",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-2": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Les signes d'un manque",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-3": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Comment se rééquilibrer",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    }
  },
  "order": ["main", "accordion-1", "accordion-2", "accordion-3"]
}
```

- [ ] **Step 2: Create `templates/page.info-richesse.json`**

Identical structure; only the three `title` values differ:

```json
{
  "sections": {
    "main": {
      "type": "main-page",
      "settings": {
        "padding_top": 28,
        "padding_bottom": 16
      }
    },
    "accordion-1": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Une eau unique au monde",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-2": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Des bienfaits prouvés",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-3": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Une pureté préservée",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    }
  },
  "order": ["main", "accordion-1", "accordion-2", "accordion-3"]
}
```

- [ ] **Step 3: Create `templates/page.info-rituel.json`**

```json
{
  "sections": {
    "main": {
      "type": "main-page",
      "settings": {
        "padding_top": 28,
        "padding_bottom": 16
      }
    },
    "accordion-1": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Le matin",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-2": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Autour de l'effort",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    },
    "accordion-3": {
      "type": "info-accordion",
      "blocks": {
        "row-1": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } },
        "row-2": { "type": "row", "settings": { "text": "<p>Ajoutez votre texte ici.</p>" } }
      },
      "block_order": ["row-1", "row-2"],
      "settings": {
        "title": "Le soir",
        "color_scheme": "scheme-1",
        "padding_top": 8,
        "padding_bottom": 8
      }
    }
  },
  "order": ["main", "accordion-1", "accordion-2", "accordion-3"]
}
```

- [ ] **Step 4: Theme check**

```bash
cd /home/pwaps/odevie && shopify theme check --fail-level error 2>&1 | tail -25
```

Expected: still only the 2 known `sections/main-product.liquid` errors; nothing about `templates/page.info-*.json`.

- [ ] **Step 5: Start the dev server**

```bash
pgrep -fa "run.js theme dev" || true   # another session may own an instance — reuse it if so
cd /home/pwaps/odevie && shopify theme dev > /tmp/theme-dev.log 2>&1 &
sleep 25 && grep -o 'http://127.0.0.1:[0-9]*' /tmp/theme-dev.log | head -1
```

Expected: a URL like `http://127.0.0.1:34787`. Export it: `PORT=<port from log>`. (Auth is cached for test-odevie.myshopify.com; the port is dynamic, never assume 9292.)

- [ ] **Step 6: Find a page to render the templates against**

The templates render at `/pages/<handle>?view=<template-suffix>`. Probe for an existing page:

```bash
for h in mineraux richesse rituel contact; do
  echo -n "$h: "; curl -s -o /dev/null -w "%{http_code}\n" "http://127.0.0.1:$PORT/pages/$h"
done
```

Expected: at least one `200`.
- If `mineraux`/`richesse`/`rituel` return 200, the store pages already exist — use them directly (no `?view=` needed since their template is assigned in admin; if not yet assigned, still use `?view=`).
- If only `contact` (or another handle) returns 200, use it with `?view=`: e.g. `http://127.0.0.1:$PORT/pages/contact?view=info-mineraux`.
- **If everything 404s: STOP and ask the human** to create the three pages in the Shopify admin (Manual follow-up section below), then re-run this step.

- [ ] **Step 7: Verify the rendered template**

Using whichever URL Step 6 produced (example uses `contact?view=info-mineraux`):

```bash
curl -s "http://127.0.0.1:$PORT/pages/contact?view=info-mineraux" -o /tmp/info-mineraux.html
grep -c "Liquid error" /tmp/info-mineraux.html
grep -o 'odevie-info-accordion__details' /tmp/info-mineraux.html | wc -l
grep -o 'odevie-info-accordion__text' /tmp/info-mineraux.html | wc -l
grep -o 'Ajoutez votre texte ici' /tmp/info-mineraux.html | wc -l
grep -c 'Le rôle des minéraux' /tmp/info-mineraux.html
grep -o '<h1[^>]*>' /tmp/info-mineraux.html | head -1
```

Expected, in order: `0` (no Liquid errors) · `3` (three drop-downs) · `6` (two text rows × 3) · `6` · `1` or more (first drop-down title present) · one `<h1 …>` tag (page title from main-page). Repeat the curl + `grep -c "Liquid error"` for `?view=info-richesse` and `?view=info-rituel` (expect `0` each, and one distinctive title each: `Une eau unique au monde`, `Le matin`).

- [ ] **Step 8: Commit**

```bash
cd /home/pwaps/odevie && git add templates/page.info-mineraux.json templates/page.info-richesse.json templates/page.info-rituel.json && git commit -m "Add info page templates: mineraux, richesse, rituel

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: Feature-grid links + final end-to-end check

**Files:**
- Modify: `templates/index.json:48-50` (the three feature-grid card blocks)

**Interfaces:**
- Consumes: page handles `mineraux`, `richesse`, `rituel` (Task 2 / admin setup); `sections/feature-grid.liquid` already renders `block.settings.link` as the card `href` — no section change.
- Produces: homepage cards linking to `/pages/mineraux`, `/pages/richesse`, `/pages/rituel`.

- [ ] **Step 1: Edit the three card links in `templates/index.json`**

Replace the three `link` values (currently `"shopify://collections/all"`) in the `feature-grid` section's blocks — card order matches the handles:

```json
"card-1": { "type": "card", "settings": { "title": "Pourquoi sommes-nous si nombreux à manquer de minéraux ?", "link": "/pages/mineraux" } },
"card-2": { "type": "card", "settings": { "title": "La richesse naturelle de l'eau de mer, prouvée et préservée", "link": "/pages/richesse" } },
"card-3": { "type": "card", "settings": { "title": "Un rituel simple dans votre quotidien", "link": "/pages/rituel" } }
```

Nothing else in `index.json` changes.

- [ ] **Step 2: Verify the rendered homepage**

Dev server should still be running from Task 2 (if not, restart per Task 2 Step 5).

```bash
curl -s "http://127.0.0.1:$PORT/" -o /tmp/home.html
grep -c "Liquid error" /tmp/home.html
grep -o 'href="/pages/\(mineraux\|richesse\|rituel\)"' /tmp/home.html | sort | uniq -c
```

Expected: `0`, then exactly one line each for `href="/pages/mineraux"`, `href="/pages/richesse"`, `href="/pages/rituel"`.

- [ ] **Step 3: Full lint pass**

```bash
cd /home/pwaps/odevie && shopify theme check --fail-level error 2>&1 | tail -25
```

Expected: only the 2 known pre-existing `sections/main-product.liquid` errors.

- [ ] **Step 4: Stop the dev server**

Only if this session started it (check ownership first):

```bash
pgrep -fa "run.js theme dev" && pkill -f "run.js theme dev"
```

(`pkill -f "shopify theme dev"` does NOT match the node process and would kill your own shell wrapper — use the `run.js` pattern.)

- [ ] **Step 5: Commit**

```bash
cd /home/pwaps/odevie && git add templates/index.json && git commit -m "Link feature-grid cards to the three info pages

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

## Manual follow-up (one-time, Shopify admin — not automatable from theme code)

In the Shopify admin (test-odevie.myshopify.com → Online Store → Pages):

1. Create three pages with **empty body content**, titled e.g. "Les minéraux", "La richesse de l'eau de mer", "Un rituel simple" — titles render as each page's `<h1>` and are merchant-editable later.
2. Set their handles to exactly `mineraux`, `richesse`, `rituel` (Edit website SEO → URL handle).
3. Assign each page its template: `info-mineraux`, `info-richesse`, `info-rituel` (Theme template dropdown).

Until this is done the homepage card links 404 — acceptable during development, prerequisite for go-live. Real content (titles, paragraphs, images per drop-down) is then entered per page in the theme editor.
