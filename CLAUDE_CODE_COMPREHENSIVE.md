# Claude Code — Comprehensive Guide Update
# Run from: ~/Documents/danielflugger
# Both guides: ai-studio-guide.html and enterprise-stack-guide.html
#
# READ THIS FIRST:
# The two guides form a sequence:
#   ai-studio-guide.html     = prototyping, AI Studio, block-by-block methodology
#   enterprise-stack-guide.html = production architecture, Claude Code, GCP depth
# Every change below reinforces this relationship.

---

## TASK 1 — ai-studio-guide.html: text fix

Find exactly:
"This is not hypothetical — it is the exact architecture described in the product plan and the exact project you will execute on danielflugger.com."

Replace with:
"This is a minimal instructions stack for a new project on a simple cms project with gated content."

---

## TASK 2 — ai-studio-guide.html: fix subtitle

Find in the hero/header section:
"Beyond vibe coding — how to build production-grade systems block by block"

Replace with:
"A disciplined methodology for building production-grade systems block by block"

Also find and replace ALL remaining instances of "vibe coding", "vibe-coding",
"vibe code", "Vibe Coding" with "rapid prototyping" or "natural language development"
as appropriate. Exception: keep the official Google codelab link text unchanged.

---

## TASK 3 — ai-studio-guide.html: fix section ordering

Currently Section 00 (End-to-End Walkthrough) appears BEFORE Section 01 (Why AI Studio).
This is wrong for a first-time reader — they hit build instructions before context.

Reorder the sections so the page reads:
01 — Why AI Studio in 2026  (currently exists, move to top)
02 — The Project Dashboard Pattern
03 — Style Guide Dashboard
04 — GCP Stack Reference Card
05 — The Project Evolution MD
06 — System Instructions as Onboarding Packages
07 — Build Block by Block  ← ADD the Plumbing→Soil→House sequence here (see Task 6)
08 — Working with the Gemini Developer API
09 — Version Control Strategy
10 — Advanced Prompting Patterns
11 — From Prototype to Production on Cloud Run
12 — The Verification Layer
00 — End-to-End: Starting From Zero  ← move to LAST as the capstone walkthrough

Update the TOC to match the new order.
Renumber the section-num labels to match.

---

## TASK 4 — ai-studio-guide.html: add "Start Here" callout and cross-link

At the very top of the page, immediately after the meta-row (Daniel Flügger · March 2026 · Google Cloud Partner), add a callout box with this content:

"This guide covers prototyping, project methodology, and AI Studio workflows.
For production architecture, Claude Code in CI/CD, multi-agent patterns, and
GCP security — continue to the Enterprise Stack Guide →"

Where "Enterprise Stack Guide →" links to enterprise-stack-guide.html.

Style it as a blue callout matching the existing callout style on the page.

Also update the meta-row "Companion to GDG Codelab ↗" line to read:
"Companion to GDG Codelab ↗  ·  Enterprise Stack Guide →"
Where "Enterprise Stack Guide →" links to enterprise-stack-guide.html.

---

## TASK 5 — ai-studio-guide.html: add Firebase Auth identity layer section

Add a new section after the current "Why AI Studio in 2026" section (Section 01).
Title: "The Identity Layer — Firebase Auth + GCP IAM"
Section label: "02 — Identity"

Content to write:

Opening paragraph:
Before any block is built, the identity layer must be in place. Firebase Auth is the
universal entry point for user-facing applications — it handles authentication
(email, Google, GitHub OAuth) and issues JWTs that your backend validates. The
integration with GCP IAM is what makes it production-grade: Firebase Auth tokens
can be verified server-side using Google's public keys, and Cloud Run services can
be configured to require valid tokens before accepting requests.

Include a code block labeled "Firebase Auth → Cloud Run verification — Python":
```python
from google.oauth2 import id_token
from google.auth.transport import requests as google_requests
import os

FIREBASE_PROJECT_ID = os.environ.get("GCP_PROJECT_ID")

def verify_firebase_token(id_token_str: str) -> dict:
    """
    Verify a Firebase Auth JWT token.
    Returns the decoded token claims if valid.
    Raises ValueError if invalid or expired.
    """
    try:
        decoded = id_token.verify_firebase_token(
            id_token_str,
            google_requests.Request(),
            audience=FIREBASE_PROJECT_ID
        )
        return decoded  # contains uid, email, and any custom claims
    except Exception as e:
        raise ValueError(f"Invalid Firebase token: {e}")

# FastAPI middleware usage
from fastapi import Request, HTTPException

async def require_auth(request: Request) -> dict:
    """Dependency — validates Firebase token on every protected route."""
    auth_header = request.headers.get("Authorization", "")
    if not auth_header.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Missing auth token")
    token = auth_header.split("Bearer ")[1]
    return verify_firebase_token(token)
```

Add a callout (amber) with label "DASHBOARD.md addition":
"Add to your DASHBOARD.md § Identity: Firebase project ID, whether custom claims
are in use, and which routes require auth vs are public. The identity layer is
the first thing a new engineer needs to understand and the last thing to be
documented in most projects."

Add a callout (blue) with label "IAM integration":
"Firebase Auth UIDs can be mapped to GCP IAM conditions using IAM Conditions.
This lets you grant specific GCP resource access to specific authenticated users
without a separate user management system. Document the mapping in DASHBOARD.md
§ Service Accounts."

---

## TASK 6 — ai-studio-guide.html: add build sequence metaphor to Block-by-Block section

Find the existing "Build Block by Block" section.
After the existing opening paragraphs and before the step-list, add this subsection:

Heading (h3): "The construction sequence"

Paragraph:
The sequence below maps directly to how physical construction works — and for
the same reason. You cannot hang windows before the walls are framed. You cannot
lay furniture before the floors are finished. In software, skipping the sequence
produces the same result: expensive rework on a foundation that cannot support
what you tried to build on it.

Then add a step-list with these five steps (matching the existing step-list style):

Step 01: "Plumbing — IAM, APIs, service accounts"
Description: Enable every GCP API you will use. Create service accounts with
minimum viable roles. Set up Secret Manager. Verify credentials before touching
any application code. Nothing built on top of this layer should need to change
IAM permissions later — if it does, the plumbing was wrong.

Step 02: "Soil — Schema-first database design"
Description: Define your database schema before any application code. Tables,
columns, indexes, constraints, and for spatial projects: PostGIS extension and
geometry columns with explicit SRID. Schema-first design prevents the most
expensive form of technical debt: a data model that has to be migrated after
application logic has been written against it. CREATE the tables. Verify them.
Only then move to the next layer.

Step 03: "House — API surface and business logic"
Description: Build the FastAPI routes and service functions against the
schema you just created. Not the UI, not the AI layer — the typed API surface
that everything else will call. Every route validated with Pydantic. Every
database call going through db.py. The house is the structure. It must be
solid before you add anything else.

Step 04: "Furniture — data, integrations, AI layer"
Description: Migrate or seed real data. Add the LLM calls — always in the
service layer, never in routes, never touching the database directly.
Add any third-party integrations (payment, auth, external APIs). Each
piece of furniture is independently removable without touching the structure.

Step 05: "Windows — frontend and UI"
Description: The UI is the last thing you build. Always. A beautiful UI
over broken logic is the most common and most expensive mistake in AI Studio
prototyping. When the logic layer is deployed, tested against real Cloud Run
infrastructure, and returning correct responses — then open AI Studio's Build
tab and build the frontend.

---

## TASK 7 — enterprise-stack-guide.html: add "Start Here" callout and cross-link

At the very top, immediately after the meta-row, add a callout box:

"New to this stack? Start with the Google AI Studio Playbook ← for prototyping
methodology, project dashboards, and the block-by-block build sequence.
This guide picks up where that one ends."

Where "Google AI Studio Playbook ←" links to ai-studio-guide.html.

Also add to the footer of the page (before the existing footer links):
"← Google AI Studio Playbook  ·  VERA Framework →"
Both as links to ai-studio-guide.html and vera.html respectively.

---

## TASK 8 — enterprise-stack-guide.html: add Firebase Auth to Section 01 stack table

Find the stack reference table in Section 01 (The Sovereign Stack Philosophy).
It currently has rows for: Reasoning, Infrastructure, CI/CD Agent, Knowledge, Secrets.

Add a new row:
| Identity | Firebase Auth + GCP IAM | JWTs issued by Firebase, verified server-side via Google public keys; maps to GCP IAM conditions for resource-level access control |

---

## TASK 9 — enterprise-stack-guide.html: add Agentic Handshake section

Add a new section after Section 03 (Claude Code — CI/CD and Headless Mode).
Title: "Agentic Handshakes — How Agents Auth into Your Project"
Section label: "03b — Auth Pattern"
Add to TOC: "03b Agentic Handshakes — How agents authenticate without secrets"

Content:

Opening paragraph:
When Claude Code or a Gemini agent needs to operate on your GCP project,
the authentication pattern matters as much as the code it generates. An agent
with the wrong credentials — too broad, too narrow, or using long-lived JSON
keys — is either a security liability or a source of permission failures that
are slow to diagnose. The pattern below gives agents exactly what they need,
nothing more, and uses short-lived tokens rather than keys.

Add a subsection h3: "The three-level auth hierarchy"

Add a table with columns: Agent Type | Auth Method | GCP Permissions | Use Case
Rows:
- Interactive Claude Code (dev) | gcloud ADC impersonating claude-code-agent SA | BigQuery read, Storage read | Local development, read-only GCP access
- Headless Claude Code (CI/CD) | Workload Identity Federation via GitHub OIDC | Storage write, Artifact Registry write | Automated builds, deployments
- Production Cloud Run service | Attached service account (api-runner SA) | Cloud SQL, Secret Manager, Vertex AI | Runtime API calls, database access

Add a code block labeled "Dev setup — impersonate scoped SA for Claude Code":
```bash
# Give Claude Code the exact permissions it has in production — no more
gcloud auth application-default login \
  --impersonate-service-account=claude-code-agent@PROJECT_ID.iam.gserviceaccount.com

# Verify what Claude Code can and cannot do
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:claude-code-agent" \
  --format="table(bindings.role)"

# Add to CLAUDE.md so every session knows its permission boundary:
# "Auth: impersonating claude-code-agent SA — BigQuery read, Storage read only
#  Cannot: write to production DB, deploy services, access secrets"
```

Add a code block labeled "Specific prompt pattern for Claude Code — auth-aware tasks":
```
"Before making any GCP API call, check DASHBOARD.md § Service Accounts
to confirm the active SA has the required role listed.

If the task requires a permission not in the SA's role list:
- Do NOT attempt the call
- Add a TODO comment: # NEEDS_PERMISSION: roles/[required-role]
- Continue with the parts of the task that are within scope
- Report the permission gap at the end of the session

Never attempt to grant permissions or modify IAM from within a session."
```

Add a callout (amber) label "EVOLUTION.md requirement":
"Every time a new IAM role is granted to any service account, add a dated
entry to EVOLUTION.md § Architecture Decisions with: what role was granted,
to which SA, why it was needed, and what the minimum viable alternative was.
IAM drift — roles accumulating without documentation — is the most common
security debt in long-running GCP projects."

---

## TASK 10 — enterprise-stack-guide.html: expand PostGIS + VERA formal linkage

Find Section 09 (PostGIS + LLMs) or equivalent spatial section.
After the existing PostGIS initialization checklist, add:

Subsection h3: "Schema-First is non-negotiable for spatial data"

Paragraph:
Spatial columns are not like other columns. A geometry column without an
explicit SRID, without a GIST index, or with the wrong coordinate system
produces results that are wrong in ways that are hard to detect — proximity
queries that return nothing, distance calculations that are off by orders
of magnitude, or joins that silently exclude valid records. These errors
cannot be fixed by application code. They require schema changes, data
migration, and re-indexing. Schema-first design prevents all of this. Define
the spatial schema completely — SRID, geometry type, index — before any
application layer reads from it.

Add a code block labeled "Idempotent PostGIS schema — safe to run multiple times":
```sql
-- Run at deploy time, every time. Idempotent by design.
-- Log the PostGIS version in DASHBOARD.md after first run.

-- Enable extension (idempotent)
CREATE EXTENSION IF NOT EXISTS postgis;

-- Verify version — log this in DASHBOARD.md § Active Services
SELECT PostGIS_Version();

-- Create spatial table with explicit SRID (idempotent via IF NOT EXISTS)
CREATE TABLE IF NOT EXISTS properties (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name        TEXT NOT NULL,
    address     TEXT,
    property_type TEXT,
    geom        geometry(Point, 4326),  -- WGS84, always explicit
    created_at  TIMESTAMPTZ DEFAULT NOW()
);

-- GIST index — create before loading data, not after
CREATE INDEX IF NOT EXISTS idx_properties_geom
    ON properties USING GIST(geom);

-- Verify the index exists
SELECT indexname, indexdef
FROM pg_indexes
WHERE tablename = 'properties' AND indexname = 'idx_properties_geom';
```

Add a callout (blue) label "EVOLUTION.md entry template for spatial schema":
```
[DATE] Schema: properties table with PostGIS geometry
Decision: geometry(Point, 4326) with GIST index
SRID: 4326 (WGS84) — matches GPS coordinates and all public municipal datasets
Index: GIST created before data load — enables ST_DWithin index use
Alternatives: geography type (rejected: less flexible for joins),
              no SRID specified (rejected: produces wrong distance results silently)
Idempotency: all DDL uses IF NOT EXISTS — safe to run in CI/CD migrations
Revisit: if switching to 3D geometry or non-point types
```

---

## TASK 11 — enterprise-stack-guide.html: add streaming in production section

Find Section 02 (Anthropic SDK — Python Client Setup).
After the prompt caching subsection, add:

Subsection h3: "Streaming responses in production"

Paragraph:
Streaming is the correct pattern for any response the user waits for — it
reduces perceived latency dramatically. It is also the pattern most likely to
fail silently in production: a stream that drops mid-response, a client that
disconnects, or a timeout that triggers before the stream completes. These
failure modes require different handling than a standard API error.

Add a code block labeled "Streaming with error handling and partial recovery":
```python
import anthropic
from typing import Generator

def stream_completion(
    client: anthropic.Anthropic,
    system: str,
    user: str,
    model: str = "claude-sonnet-4-6",
) -> Generator[str, None, None]:
    """
    Stream a completion. Yields text chunks as they arrive.
    Handles mid-stream failures gracefully — logs partial output,
    raises a typed exception the caller can handle.
    """
    partial_output = []
    try:
        with client.messages.stream(
            model=model,
            max_tokens=2048,
            system=system,
            messages=[{"role": "user", "content": user}],
        ) as stream:
            for text in stream.text_stream:
                partial_output.append(text)
                yield text
    except anthropic.APIStatusError as e:
        # Log partial output before raising — partial responses have value
        log.warning(
            "stream_interrupted",
            extra={
                "partial_chars": sum(len(t) for t in partial_output),
                "error": str(e)
            }
        )
        raise StreamInterruptedError(
            f"Stream failed after {len(partial_output)} chunks: {e}",
            partial="".join(partial_output)
        ) from e

class StreamInterruptedError(Exception):
    """Stream failed mid-response. partial attribute contains what arrived."""
    def __init__(self, message: str, partial: str = ""):
        super().__init__(message)
        self.partial = partial

# FastAPI SSE endpoint pattern
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

@app.get("/stream/{query}")
async def stream_response(query: str):
    """Server-Sent Events endpoint for streaming Claude responses."""
    def event_generator():
        for chunk in stream_completion(client, SYSTEM_PROMPT, query):
            yield f"data: {chunk}\n\n"
        yield "data: [DONE]\n\n"
    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

Add a callout (amber) label "Do not stream in AI Studio preview":
"AI Studio's Build tab preview breaks with streaming responses — the iframe
cannot handle SSE. Prototype all streaming features with non-streaming calls
in AI Studio, switch to streaming only when deploying to Cloud Run. Log this
in EVOLUTION.md § What Was Tried and Abandoned."

---

## TASK 12 — enterprise-stack-guide.html: add cost attribution per feature

Find Section 07 (Cost Engineering).
After the model routing section, add:

Subsection h3: "Cost attribution — tag every request"

Paragraph:
Token cost without attribution is useless for optimization. Knowing your
monthly Claude bill is $800 tells you nothing about which feature to optimize.
Knowing that your document classification feature costs $340/month and your
summary generation costs $460/month tells you exactly where to focus. One
metadata field on every API call makes this possible.

Add a code block labeled "Cost attribution — metadata tagging pattern":
```python
import anthropic
import logging
from dataclasses import dataclass

log = logging.getLogger(__name__)

@dataclass
class RequestContext:
    """Tag every LLM call with context for cost attribution."""
    feature: str       # "document_classification", "summary_generation", etc.
    user_id: str       # for per-user cost tracking
    request_type: str  # "classify", "generate", "extract", "reason"
    environment: str   # "prod", "staging", "dev"

def tracked_complete(
    client: anthropic.Anthropic,
    system: str,
    user: str,
    model: str,
    ctx: RequestContext,
) -> str:
    """Complete with full cost attribution logging."""
    response = client.messages.create(
        model=model,
        max_tokens=2048,
        system=system,
        messages=[{"role": "user", "content": user}],
    )

    # Log structured cost data — queryable in Cloud Logging
    cost_usd = (
        response.usage.input_tokens * 0.000003 +   # Sonnet input price
        response.usage.output_tokens * 0.000015    # Sonnet output price
    )
    log.info("llm_cost", extra={
        "feature": ctx.feature,
        "user_id": ctx.user_id,
        "request_type": ctx.request_type,
        "environment": ctx.environment,
        "model": model,
        "input_tokens": response.usage.input_tokens,
        "output_tokens": response.usage.output_tokens,
        "cache_read_tokens": getattr(response.usage, "cache_read_input_tokens", 0),
        "estimated_cost_usd": round(cost_usd, 6),
    })
    return response.content[0].text

# Usage — cost is visible and attributable at the call site
result = tracked_complete(
    client, system, user, MODEL_SONNET,
    ctx=RequestContext(
        feature="document_classification",
        user_id=user_id,
        request_type="classify",
        environment="prod"
    )
)
```

Add a callout (green) label "BigQuery cost dashboard":
"Structured logs flow to Cloud Logging automatically on Cloud Run. Export
to BigQuery with a log sink and you have a queryable cost dashboard:
SELECT feature, SUM(estimated_cost_usd) as total_cost
FROM llm_cost_logs GROUP BY feature ORDER BY total_cost DESC.
One Cloud Logging export, zero additional instrumentation."

---

## TASK 13 — enterprise-stack-guide.html: strengthen VERA formal linkage

Find the existing VERA / Verification section (last section).
Replace or expand the existing content to include:

After the existing opening paragraphs, add:

Subsection h3: "Linking every architectural decision to EVOLUTION.md"

Paragraph:
The VERA framework is not a retrospective exercise. Every architectural decision
in this guide — Firebase Auth over custom auth, Cloud Run over Cloud Functions,
ST_DWithin over ST_Distance, WIF over JSON keys — should have a dated
EVOLUTION.md entry at the moment it is made. Not after the project ships. Not
in a documentation sprint. At the moment the decision is made, while the
reasoning is fresh and the alternatives are still visible.

Add a code block (MD sample style) labeled "EVOLUTION.md — required entry format for every architectural decision":
```markdown
## 02. Architecture Decisions

# Format: [DATE] DECISION | alternatives considered + rejected | cost/risk | revisit trigger

[2026-04-07] Auth: Firebase Auth + Cloud Run verification
Decision: Firebase Auth JWTs verified server-side via google-auth library
Alternatives:
  - Custom JWT (rejected: key rotation complexity, no OAuth provider integration)
  - Supabase Auth (rejected: adds platform dependency outside GCP ecosystem)
  - No auth on public endpoints (rejected: gated content requires identity)
Cost: Zero additional GCP cost — verification uses Google's public keys
IAM impact: Cloud Run services remain --no-allow-unauthenticated for protected routes
Revisit when: Firebase Auth pricing changes or enterprise SSO requirement emerges

[2026-04-07] Spatial: ST_DWithin over ST_Distance for proximity queries
Decision: All proximity queries use ST_DWithin(geom::geography, point, radius)
Alternatives:
  - ST_Distance in WHERE clause (rejected: does not use GIST index, full table scan)
  - Application-side bounding box filter (rejected: imprecise, more code)
Performance: <50ms on 100k rows with GIST index vs >2000ms without
Idempotency: no migration needed, query-level decision
Revisit: Never. This is a PostGIS fundamental.

[2026-04-07] Deployment: Cloud Run over Cloud Functions
Decision: All backend services on Cloud Run (containerized FastAPI)
Alternatives:
  - Cloud Functions (rejected: 9MB unzipped limit breaks most Python deps,
    no persistent connections, cold start per-invocation model)
  - GKE (rejected: operational overhead, cost at this scale)
  - App Engine (rejected: deprecated patterns, less flexible runtime)
Cost: Per-request billing, min-instances=0 for dev, min-instances=1 for prod
Revisit when: request volume exceeds ~10M/month (GKE cost crossover)
```

Add a callout (blue) label "VERA certification path":
"The EVOLUTION.md pattern described here is the foundation of the VERA
maturity model. An organization with consistently documented architecture
decisions, verified outputs, and sovereign knowledge infrastructure is at
VERA maturity level 3. The formal framework, including the Ed25519 session
certificate methodology and multi-party attestation tier, is at
veraframework.com and danielflugger.com/vera.html."

---

## TASK 14 — Both guides: update nav to full sitewide nav

Both ai-studio-guide.html and enterprise-stack-guide.html have non-standard navs.
Replace the nav in both files with the full sitewide nav from the-restack.html,
including the Experience → Portfolio dropdown and the Learn dropdown with both guides.

Match exactly — same classes, same structure, same items.

---

## DEPLOY

After all tasks complete and verified locally:

```bash
cd ~/Documents/danielflugger
git add -A
git commit -m "feat: comprehensive guide update — cross-links, Firebase Auth, streaming, cost attribution, VERA formal linkage, PostGIS schema-first, agentic handshakes, build sequence metaphor"
git push origin main && firebase deploy
```

Post-deploy verification:
- danielflugger.com/ai-studio-guide.html
  ✓ No "vibe coding" in subtitle or body
  ✓ Section order: 01 Why first, 00 Walkthrough last
  ✓ "Start Here" callout links to enterprise-stack-guide.html
  ✓ Firebase Auth section present
  ✓ Plumbing→Soil→House→Furniture→Windows sequence in Block-by-Block
  ✓ Nav matches the-restack.html

- danielflugger.com/enterprise-stack-guide.html
  ✓ "Start Here" callout links to ai-studio-guide.html
  ✓ Firebase Auth row in stack table
  ✓ Agentic Handshakes section (03b) in nav and content
  ✓ Streaming in production code block present
  ✓ Cost attribution with BigQuery export pattern present
  ✓ EVOLUTION.md required entry format present
  ✓ Nav matches the-restack.html
  ✓ Footer links: ← AI Studio Playbook · VERA Framework →

- Any page — cross-links resolve correctly in both directions
