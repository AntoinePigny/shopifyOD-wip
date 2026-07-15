# Info pages — manual follow-up checklist

The `feature/info-pages` branch is complete and review-approved (commits `50fccaa0..3b2ad879`). These last steps can't be done from theme code — they happen in the Shopify admin or are your call.

## 1. Create the three pages in the Shopify admin

In **test-odevie.myshopify.com admin → Online Store → Pages**, create three pages:

| Page title (suggestion, shows as the page's H1) | URL handle (exact) | Theme template |
|---|---|---|
| Les minéraux | `mineraux` | `info-mineraux` |
| La richesse de l'eau de mer | `richesse` | `info-richesse` |
| Un rituel simple | `rituel` | `info-rituel` |

For each page:
- [ ] Leave the **body content empty** (the drop-downs come from the template, not the page body)
- [ ] Set the **URL handle** exactly as above (Edit website SEO → URL handle)
- [ ] Pick the matching **Theme template** in the right-hand sidebar

Until this is done, the three homepage feature-grid cards link to 404s — expected during development, blocking for go-live.

## 2. Check the theme editor experience

Open one of the pages in the theme editor (Online Store → Customize → pick the page from the top selector):

- [ ] Clicking a drop-down section (or one of its Row blocks) in the left sidebar should **auto-open** that drop-down in the preview, and it should stay open while you edit — this is the design-mode script that couldn't be tested from the CLI
- [ ] Enter real content: per drop-down, edit the title, edit/add Row blocks (paragraph + image each; desktop alternates image right/left automatically)

## 3. Merge when ready

- [ ] Merge `feature/info-pages` into `main` (or ask Claude to do it)

## Optional (flagged by review, zero urgency)

- [ ] In the theme editor, re-pick the three feature-grid card links via the link picker so they're stored as `shopify://pages/…` instead of raw `/pages/…` paths — future-proofs them if the store ever adds languages. No code change needed.

## resume the session with 
 - claude --resume 623bfa46-7c3c-46d9-9554-da47427d1584