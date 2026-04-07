# Claude Code Session — Final Style Alignment
# Run from: ~/Documents/danielflugger
# This is a STYLE-ONLY task. Do not change any content, links, or nav.

---

## CONTEXT — what the screenshots show

**enterprise-stack-guide.html** — background and typography are CORRECT. This is
the target for background color. Do not touch this file except for typography
matching in Task 2.

**the-restack.html** — body typography (font faces, sizes, colors, line-height,
paragraph spacing) is CORRECT. This is the typography reference. Do not touch
this file at all.

**authority.html, rank-yourself-first.html, the-proof-economy.html,
company-of-one.html** — wrong background color (warm beige). Wrong typography.
Both need to change.

**ai-studio-guide.html** — wrong body typography. Background may already be correct.

---

## BEFORE STARTING — read these values from source files

Open each file and note the EXACT values. Do not guess.

1. From `enterprise-stack-guide.html` or `style.css`:
   Find the `body` background-color. Write it down. This is TARGET_BG.

2. From `the-restack.html` or `style.css`:
   Find these body typography values and write them down:
   - body font-family
   - body font-size
   - body font-weight
   - body line-height
   - body color (text color)
   - paragraph (p) font-size, line-height, color, margin-bottom
   - h1 font-family, font-size, font-weight, color, line-height
   - h2 font-family, font-size, font-weight, color, line-height
   - h3 font-family, font-size, font-weight, color
   - Any section-label / eyebrow text styles (the small caps "OPENING", "Essay · 2026" labels)

   These are your TYPOGRAPHY_REFERENCE values.

---

## TASK 1 — Background color: apply TARGET_BG to 5 essay pages

Apply the background-color value from enterprise-stack-guide.html to:
- authority.html
- rank-yourself-first.html
- the-proof-economy.html
- company-of-one.html
- ai-studio-guide.html (if different from TARGET_BG)

Each of these pages has either:
a) An inline `<style>` block with a `body { background-color: ... }` rule — update that value
b) A class or element style that sets the background — find and update it
c) The background set via style.css — in that case update only the value in style.css
   that applies to these pages, without affecting enterprise-stack-guide.html

After updating: grep the project for the old warm beige hex value (likely #f3f1ec or
similar) to confirm zero remaining instances on these pages.

---

## TASK 2 — Body typography: apply TYPOGRAPHY_REFERENCE to 6 pages

Apply the exact typography values from the-restack.html to the BODY CONTENT of:
- authority.html
- rank-yourself-first.html
- the-proof-economy.html
- company-of-one.html
- ai-studio-guide.html
- enterprise-stack-guide.html

**What to match exactly:**
- body font-family, font-size, font-weight, line-height, color
- p (paragraph) font-size, line-height, color, margin-bottom, font-weight
- h1 — font-family, font-size, font-weight, line-height, color
- h2 — font-family, font-size, font-weight, line-height, color
- h3 — font-family, font-size, font-weight, color
- Section label / eyebrow styles (the small monospace labels like "OPENING", "01 — Foundation")

**How to apply it:**
- If the page uses style.css and the values are already in style.css correctly:
  confirm the page is using those classes, no action needed
- If the page has an inline `<style>` block overriding style.css:
  update the inline values to match TYPOGRAPHY_REFERENCE
- If the page has inline style="" attributes on elements:
  update those values

**Do NOT change:**
- Nav styles
- Code block styles (the dark background <pre> blocks in the guide pages)
- Dashboard card styles in guide pages
- Callout box styles in guide pages
- Table styles in guide pages
- Any layout/grid/flexbox properties
- Any colors that are intentional accent colors (blue callouts, green badges etc.)
- Link colors in the body text if they have a specific accent treatment

**For ai-studio-guide.html and enterprise-stack-guide.html specifically:**
The guide pages have extensive inline CSS for their unique components (code blocks,
step lists, tables, callout boxes). Only update the base body typography properties:
body, p, h1, h2, h3, and the eyebrow/section-label styles.
Leave all component-specific styles untouched.

---

## VERIFY before deploying

For each of the 6 pages, confirm:
1. Body background matches enterprise-stack-guide.html
2. Body paragraph text — font, size, color matches the-restack.html
3. Headings — font, size, color matches the-restack.html
4. No layout broke (check nav, content columns, footer)
5. Code blocks in guide pages still have dark background (they should be unaffected)

---

## DEPLOY

```bash
cd ~/Documents/danielflugger
git add -A
git commit -m "style: body background and typography alignment across all research and learn pages"
git push origin main && firebase deploy
```

Post-deploy spot checks:
- danielflugger.com/authority.html — background white/light, body text matches the-restack
- danielflugger.com/the-proof-economy.html — same
- danielflugger.com/company-of-one.html — same
- danielflugger.com/enterprise-stack-guide.html — unchanged background, body text matches the-restack
- danielflugger.com/ai-studio-guide.html — background + body text matches the-restack
