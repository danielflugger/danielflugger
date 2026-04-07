# Claude Code Session — danielflugger.com Updates
# Run from: ~/Documents/danielflugger
# Deploy: git add -A && git commit -m "message" && git push origin main && firebase deploy

---

## READ FIRST — understand these files before touching anything

1. `style.css` — the site stylesheet, used by all pages except the inline-styled essays
2. `the-restack.html` — the style reference for all research/learn pages
3. `index.html` — homepage (dark navy nav, different from essay pages)
4. `ai-studio-guide.html` — guide to update (currently has self-contained CSS)
5. `enterprise-stack-guide.html` — guide to update (self-contained CSS)

Check the Writing dropdown in `the-restack.html` nav to understand the existing
dropdown pattern before adding new dropdowns. Match it exactly.

---

## TASK 1 — Update style.css: add new dropdown nav items

Find the existing `.dropdown` or nav CSS in style.css.
Add support for:

**Experience dropdown:**
- "Portfolio" → portfolio.html

**New top-level item: Learn**
- "Enterprise Stack Guide" → enterprise-stack-guide.html  
- "Google AI Studio Playbook" → ai-studio-guide.html

Match the exact CSS pattern used by the existing Writing dropdown.
Do not change any other styles.

---

## TASK 2 — Update nav on ALL these pages (sitewide)

Files to update:
index.html, about.html, experience.html, portfolio.html, research.html,
vera.html, contact.html, the-proof-economy.html, authority.html,
rank-yourself-first.html, the-restack.html, company-of-one.html,
ai-studio-guide.html, enterprise-stack-guide.html

Changes to nav on every file:
1. Move Portfolio from top-level nav → dropdown under Experience
   Before: Experience (standalone) | Portfolio (standalone)
   After:  Experience (with dropdown: Portfolio → portfolio.html)

2. Add Learn as new top-level nav item with dropdown:
   - Enterprise Stack Guide → enterprise-stack-guide.html
   - Google AI Studio Playbook → ai-studio-guide.html

3. Keep all other nav items exactly as they are on each page

NOTE: index.html uses a DIFFERENT nav style (dark navy background).
Update its nav items but do not change its visual style.
All other pages use the standard light nav from style.css — keep that.

---

## TASK 3 — ai-studio-guide.html: style alignment + content updates

### 3a. Replace "vibe coding" language throughout
Find and replace every instance of these phrases with more professional alternatives:
- "vibe coding" → "rapid prototyping"
- "vibe code" → "prototype"  
- "vibe-coding" → "rapid prototyping"
- "Vibe Coding" → "Rapid Prototyping"
- "vibe-code" → "natural language development"

Exception: Keep the codelab link text unchanged if it references the official
Google codelab title (that's Google's name, not ours).

### 3b. Style alignment with the-restack.html
The guide currently has a self-contained CSS block and a custom nav.
Replace the nav HTML with the site-standard nav from the-restack.html
(including the new dropdowns from Task 2).

The HERO/TITLE section (the cream background section with Playfair Display
"Google AI Studio: The Enterprise Developer's Playbook" heading) stays
exactly as-is. Do not change fonts, colors, or layout of the hero.

For the rest of the page — the TOC, section content, code blocks —
they can keep their existing inline styles. The goal is nav consistency,
not a full redesign.

### 3c. Add these missing content items as new subsections

**In Section 08 (Gemini Developer API), add after the export code workflow:**

Subsection: "Context Caching in Gemini — 90% Cost Reduction"
- Gemini has context caching equivalent to Anthropic's prompt caching
- Cache large, repeated context (DASHBOARD.md, CLAUDE.md, schema docs)
- Minimum 32,768 tokens to be eligible (higher than Anthropic's 1024)
- Cached content charged at ~25% of normal input price after creation
- Show a minimal Python example using google.generativeai CachedContent API
- Note: cache TTL is configurable, default 1 hour

Subsection: "Grounding with Google Search"
- Enables real-time web data in Gemini responses
- Valuable for: competitive analysis, current pricing, recent API changes
- One parameter addition: tools=[{"google_search_retrieval": {}}]
- Show the minimal code change
- Note: grounded responses include source citations automatically

**In Section 10 (Advanced Prompting Patterns), add after Pattern 6:**

Pattern 7: "Temperature and cost"
- Lower temperature = fewer output tokens on average (model converges faster)
- At 0.1 on a classification task, outputs are shorter and more direct
- At 0.9 on the same task, outputs are longer and more exploratory
- Temperature is a cost lever, not just a quality lever

---

## TASK 4 — enterprise-stack-guide.html: style alignment + content additions

### 4a. Style alignment
Same as Task 3b — replace the self-contained nav with the site-standard nav
from the-restack.html including new dropdowns. Keep all content styles as-is.

### 4b. Add these missing sections

**Add to Section 07 (Cost Engineering), after the Model Routing section:**

Subsection: "Count tokens before expensive calls"
Add this code block:
```python
# Count tokens before sending — one line, prevents surprise bills
token_count = client.messages.count_tokens(
    model="claude-sonnet-4-6",
    system=system_prompt,
    messages=[{"role": "user", "content": user_prompt}]
)
print(f"This request will use ~{token_count.input_tokens} input tokens")
# At Sonnet pricing: input_tokens * $0.000003 = estimated cost
# Set a threshold: if token_count.input_tokens > 50000: use_cache_or_truncate()
```
Note: use this in development to audit expensive prompts before they hit production.

Subsection: "Extended thinking for complex architectural reasoning"
- When Sonnet's first answer isn't sufficient for complex tradeoffs, use extended thinking
- Model thinks through the problem before responding — visible reasoning chain
- Costs more tokens but dramatically better on multi-constraint problems
- Show the beta parameter syntax:
```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # how much to spend on reasoning
    },
    messages=[{"role": "user", "content": architecture_question}],
    betas=["interleaved-thinking-2025-05-14"]
)
# response.content will include thinking blocks + response blocks
thinking_block = [b for b in response.content if b.type == "thinking"][0]
```
Note: Use for architecture decisions, not for routine generation.

Subsection: "Rate limit handling — exponential backoff with jitter"
Add note: this breaks every production system eventually and nobody adds it until
it does. Show the pattern:
```python
import time, random, anthropic
from anthropic import RateLimitError

def call_with_backoff(client, **kwargs, max_retries=5):
    """Exponential backoff with jitter for rate limit handling."""
    for attempt in range(max_retries):
        try:
            return client.messages.create(**kwargs)
        except RateLimitError:
            if attempt == max_retries - 1:
                raise
            wait = (2 ** attempt) + random.uniform(0, 1)  # jitter prevents thundering herd
            print(f"Rate limited. Waiting {wait:.1f}s (attempt {attempt+1}/{max_retries})")
            time.sleep(wait)
```

**Add to Section 03 (Claude Code), after the worktrees section:**

Subsection: "Headless mode flags you need to know"
```bash
# Required for non-interactive CI/CD — skips permission prompts
claude --dangerously-skip-permissions --headless --print "run tests"

# Structured JSON output — for parsing in scripts
claude --output-format stream-json "analyze this diff"

# Combine for CI pipeline use
claude --dangerously-skip-permissions \
       --headless \
       --output-format stream-json \
       --model claude-haiku-4-5-20251001 \
       "classify the severity of issues in this diff" | jq '.content[0].text'
```
Callout (amber): --dangerously-skip-permissions gives Claude Code full file
system access without prompting. Only use in sandboxed CI environments.
Never on a machine with production credentials.

**Add to Section 12 (Security), after the IAM rules:**

Subsection: "Workload Identity Federation — no JSON keys in production"
- Service account JSON keys are a security antipattern: they're long-lived,
  can be leaked, and can't be scoped to a specific workload
- Workload Identity Federation lets GitHub Actions (or any OIDC provider)
  authenticate as a GCP service account without a key file
- Show the gcloud setup commands:
```bash
# Create the workload identity pool
gcloud iam workload-identity-pools create "github-pool" \
  --project="${PROJECT_ID}" \
  --location="global"

# Create the OIDC provider
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"

# In GitHub Actions — no secrets needed
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/PROJECT_NUM/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
    service_account: 'deploy-bot@PROJECT_ID.iam.gserviceaccount.com'
```
Note: Add to DASHBOARD.md § Service Accounts with a "uses WIF, no key file" annotation.

---

## TASK 5 — Deploy

After all changes are complete and verified locally:

```bash
cd ~/Documents/danielflugger
git add -A
git commit -m "feat: sitewide nav (Learn section + Experience dropdown), guide style alignment, content depth additions"
git push origin main && firebase deploy
```

Verify live at:
- danielflugger.com — check Experience dropdown shows Portfolio, Learn dropdown shows both guides
- danielflugger.com/enterprise-stack-guide.html — nav matches site
- danielflugger.com/ai-studio-guide.html — nav matches site, no "vibe coding" language
- danielflugger.com/the-restack.html — nav unchanged except new items

---

## Summary of file changes

| File | What changes |
|------|-------------|
| style.css | Add Experience dropdown + Learn dropdown CSS |
| index.html | Nav: Experience+Portfolio dropdown, add Learn dropdown |
| about.html | Nav: same |
| experience.html | Nav: same |
| portfolio.html | Nav: same |
| research.html | Nav: same |
| vera.html | Nav: same |
| contact.html | Nav: same |
| the-proof-economy.html | Nav: same |
| authority.html | Nav: same |
| rank-yourself-first.html | Nav: same |
| the-restack.html | Nav: same |
| company-of-one.html | Nav: same |
| ai-studio-guide.html | Nav + style alignment + vibe coding → rapid prototyping + content additions |
| enterprise-stack-guide.html | Nav + style alignment + content additions |

---

## TASK 6 — Background color: replace #f3f1ec with #fbfaf9 sitewide

Search every HTML file and style.css for the hex value `#f3f1ec` (with or without
the # prefix in CSS). Replace every instance with `#fbfaf9`.

Also check for the equivalent RGB value rgb(243, 241, 236) — replace with rgb(251, 250, 249).

Files to search: style.css and ALL .html files in the project root.
Use a global find-and-replace — do not do this file by file manually.

Verify: after replacement, grep for f3f1ec to confirm zero remaining instances.

---

## TASK 7 — Match title + TOC style on guide pages to company-of-one.html

Reference: https://danielflugger.com/company-of-one.html

The company-of-one.html page sets the standard for essay/research page typography.
Apply its exact title section and TOC styling to these pages:

**Pages to update:**
- enterprise-stack-guide.html
- ai-studio-guide.html
- All research/essay pages that do not already match:
  authority.html, rank-yourself-first.html, the-proof-economy.html,
  the-restack.html (check each — if it already matches, skip it)

**What to match from company-of-one.html:**
- The eyebrow text style (e.g. "Essay · March 2026") — font, size, color, spacing
- The `<h1>` title — font family, weight, size, line-height, color, margin
- The subtitle/tagline below the h1 — font, weight, color
- The author/date line — font, size, color, spacing
- The horizontal rule between hero and TOC
- The TOC container — background, padding, border treatment if any
- TOC heading ("Contents") — font, size, letter-spacing, text-transform
- TOC list items — font, size, line-height, bullet/marker style, indentation
- TOC anchor text — color, hover state, text-decoration treatment
- The anchor sub-descriptions on TOC items (the indented grey lines) — match exactly

For the guide pages (enterprise-stack-guide.html and ai-studio-guide.html),
the eyebrow label can remain "Advanced Guide · April 2026" but must use the
same font/size/color treatment as company-of-one.html's eyebrow.

For ai-studio-guide.html specifically: the hero background color and overall
title section layout can remain as-is — only match the typography within it.

---

## TASK 8 — rank-yourself-first.html: remove $500 offer, update closing section

Find the section "Build It Yourself. Own It Completely" near the bottom of
rank-yourself-first.html.

**Remove entirely:**
- The $500 offer box / pricing card / CTA with any dollar amount
- Any surrounding container, border, or card that holds the offer
- Any text referencing a price, package, or purchase

**Replace the closing of that section with exactly this text:**

```
Read the full framework: The Proof Economy →  ·  VERA methodology →
```

Where:
- "The Proof Economy →" links to the-proof-economy.html
- "VERA methodology →" links to vera.html

Style these two links to match the existing inline link style on the page
(same color, same hover treatment as other body links on rank-yourself-first.html).

Keep the "Build It Yourself. Own It Completely" heading and any prose above
the offer box — only remove the offer box itself and replace with the two links.

---

## UPDATED DEPLOY COMMAND

After all tasks (1–8) are complete:

```bash
cd ~/Documents/danielflugger
git add -A
git commit -m "feat: nav updates, style alignment, bg color, remove $500 offer, guide typography"
git push origin main && firebase deploy
```

Verify live:
- Any research page — confirm background is #fbfaf9 not #f3f1ec
- enterprise-stack-guide.html — title/TOC matches company-of-one.html typography
- ai-studio-guide.html — title/TOC matches company-of-one.html typography
- rank-yourself-first.html — $500 offer gone, two links in its place
- Any page — Experience shows Portfolio dropdown, Learn shows both guides
