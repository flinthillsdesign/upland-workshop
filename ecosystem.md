# Upland Workshop Ecosystem

## Philosophy

The Upland Workshop is a family of standalone, purpose-built apps for Upland Exhibits — a museum exhibit design and fabrication firm. Each app solves one problem well, stays independently deployable, and connects to the others through lightweight integrations rather than tight coupling.

The guiding principles:

- **Single-purpose apps** — Each tool does one thing. The Scheduler schedules. The Inquiry Hub manages inquiries. ODIN synthesizes. No app tries to be everything.
- **Shared identity, independent operation** — Every app can function if the others go down. The satellite apps share a visual identity and an auth database, but each has its own data store.
- **Convention over shared code** — Rather than a shared component library, the apps follow the same patterns: Netlify + Turso + esbuild + vanilla TypeScript. Copy the pattern, not a dependency.
- **Read-heavy integration** — Apps mostly read from each other. ODIN reads from everything. The Inquiry Hub enriches from Scheduler and Orders. Nobody writes to another app's database directly.
- **Right tool for the job** — The Upland Website is Python on Heroku because that's what it needs to be. The satellite apps are TypeScript on Netlify because that's what works for lightweight internal tools. Not everything has to match.

## The Apps

### Upland Website (uplandexhibits.com)
**Repo:** `flinthillsdesign/upland`
**Stack:** Python (Plain framework), PostgreSQL, Heroku
**Port:** 8443 (local)

The primary marketing website, e-commerce storefront, and order management system for Upland Exhibits. Product configurators (Montera panels, Reader Rails, Banner Stands), a Three.js exhibit builder, Stripe checkout, FedEx shipping integration, and a full order-tracking pipeline. It's a complicated app that does a lot — and it's intentionally separate from the satellite tools.

**Key role in ecosystem:** Exposes `/order/queue/?format=json` — the order pipeline data that ODIN and Scheduler consume for capacity planning and financial forecasting.

**Auth:** Standalone session-based auth via Plain framework. Has its own PostgreSQL users table. Not connected to the shared Turso auth, and won't be.

---

### ODIN (odin.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-odin`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3002 (local)

Operations Dashboard & Intelligence Network. The read-only intelligence layer that synthesizes data from every other tool plus Basecamp and Toggl. Uses Claude AI (Sonnet) to generate weekly meeting agendas, daily briefings, project summaries, and conversational Q&A. Stores a knowledge library of playbooks, lessons learned, and institutional knowledge.

**Key role in ecosystem:** The only app that reads from ALL other systems. Acts as the operational brain — what's happening across every project, every tool, every person.

**Integrations:**
- Scheduler API (capacity, timelines)
- Inquiry Hub API (lead pipeline)
- Storefront order JSON (production volume)
- Basecamp (OAuth — todos, messages, activity)
- Toggl (API token — hours by project/person)
- Dropbox (past meeting agendas as .docx)

**Auth:** Shared Turso auth DB + JWT

---

### Inquiry Hub (inquiries.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-inquiries`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3001 (local)

AI-powered customer inquiry management. Logs all incoming inquiries (email via Postmark webhook, Wufoo forms, manual entry), uses Claude Haiku to classify them (type, priority, summary), and generates contextual draft responses. Enriches inquiries with data from Scheduler, Orders, and ODIN.

**Key role in ecosystem:** Front door for new business. Catches every inquiry and gives the team AI-assisted context before responding.

**Integrations:**
- ODIN API (aggregated context for inquiry enrichment)
- Scheduler API (capacity, lead times)
- Storefront order JSON (open orders)
- Postmark (inbound/outbound email)

**Auth:** Shared Turso auth DB + JWT

---

### Scheduler (schedules.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-scheduler`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3000 (local)

Project scheduling and financial pipeline tracking. Monthly scheduling grids, design/fabrication FTE capacity planning, 18-month cash flow forecasting, and AI coaching (Claude Haiku). Supports client-facing read-only views via shareable word-based tokens (e.g., `cedar-river-42`).

**Key role in ecosystem:** The reference implementation for the Netlify + Turso + JWT stack. First app built in this pattern — auth, storage, and API patterns were copied to the others from here.

**Integrations:**
- Storefront order JSON (product order volume for FTE calculations)
- Basecamp (webhook notifications for project changes)
- Postmark (daily digest emails, weekly coaching reports)

**Auth:** Shared Turso auth DB + JWT. Also manages client tokens for external viewers.

---

### Luca & Claire (luca.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-luca-claire`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3003 (local)

AI-assisted exhibit panel design tool. Replaces the static Previews app (`flinthillsdesign/upland-previews`) with an interactive environment where curators and designers create exhibit panels, get AI help with content and visual design, and share previews with clients.

**Luca** — Layout Utility Curatorial Assistant. The content specialist. Helps with narrative flow across the exhibit, drafts body text and captions from source material, advises on content hierarchy and curatorial voice.

**Claire** — Creative Layout AI Rendering Engine. The design specialist. Handles typography, color palettes, layout composition, visual refinement, and preparing panels for client review.

Together they're two AI team members — each powered by Claude with exhibit design expertise — who overlap naturally but bring distinct specialties. Luca handles the what and where, Claire handles the how it looks.

**Key role in ecosystem:** Turns curatorial content into designed exhibit panels. Projects contain panels defined by physical dimensions and panel type. Content is structured (title, body, caption, images) and rendered as HTML/CSS. Clients view galleries via shareable token links.

**Integrations:**
- Claude API (Luca and Claire AI assistance)
- Postmark (password reset, gallery share notifications)

**Auth:** Shared Turso auth DB + JWT

---

## Shared Infrastructure

### Auth Database (Turso)
A single Turso database (`upland-auth`) shared by Scheduler, ODIN, and Inquiry Hub. Contains one `users` table:

| Field | Purpose |
|-------|---------|
| id | nanoid |
| username | Login identifier |
| password_hash | bcrypt |
| role | superadmin, manager, editor |
| name | Display name |
| email | For password reset |
| token_invalid_before | ISO timestamp — JWTs issued before this are rejected |
| can_finance | Boolean — access to financial data |

All three satellite apps share a `JWT_SECRET`. A token minted by the Scheduler is valid in ODIN and Inquiry Hub.

The Upland Website has its own auth and is not part of this system.

### Dual Database Pattern
Each satellite app connects to TWO Turso databases:
1. **Auth DB** (`TURSO_AUTH_URL`) — shared users table (read-only from most apps)
2. **App DB** (`TURSO_URL`) — app-specific tables (full CRUD)

Connection code auto-switches between `@libsql/client` (local SQLite files) and `@libsql/client/http` (remote Turso) based on whether the URL starts with `file:`.

### Schema Migration Pattern
All apps use `ensureSchema()` — idempotent `CREATE TABLE IF NOT EXISTS` + `ALTER TABLE` in try/catch. No migration files, no migration state. Every cold start verifies the schema is current.

### Shared Tech Conventions (Satellite Apps)
| Convention | Detail |
|------------|--------|
| Framework | Vanilla TypeScript (no React/Vue/Svelte) |
| Build | esbuild (TS to JS compilation) |
| Lint | oxlint |
| Format | oxfmt |
| Hosting | Netlify (functions + static) |
| Database | Turso/libsql |
| Auth | JWT + bcrypt (shared secret) |
| Email | Postmark |
| AI | Anthropic Claude API |
| IDs | nanoid |
| Font | Instrument Sans |
| Brand color | #A5BC39 |
| Background | #161A22 to #0A0D12 gradient |
| API pattern | Single Netlify function, hand-rolled router |
| Dev server | Express wrapping the function handler |

### Service-to-Service Communication
Apps call each other via HTTPS with either:
- **JWT auth** — ODIN mints a service JWT (`userId: "service-odin"`) to call Scheduler/Inquiry APIs
- **INTERNAL_SERVICE_TOKEN** — Bearer token for simpler auth
- **No auth** — Storefront order JSON endpoint is unauthenticated

### Cross-App Data Flow

```
Basecamp ──────────┐
Toggl ─────────────┤
                   v
            ┌─────────────┐
            │    ODIN      │ <── reads from everything
            │  (AI brain)  │
            └──────┬──────┘
                   │ provides enrichment context
                   v
         ┌─────────────────┐     ┌───────────────────┐
         │  Inquiry Hub    │────>│    Postmark        │
         │ (front door)    │     │   (email in/out)   │
         └────────┬────────┘     └───────────────────┘
                  │ checks capacity
                  v
         ┌─────────────────┐     ┌───────────────────┐
         │   Scheduler     │────>│   Basecamp         │
         │ (planning)      │     │  (webhook notify)  │
         └────────┬────────┘     └───────────────────┘
                  │ reads order volume
                  v
         ┌─────────────────┐
         │ Upland Website  │ <── standalone, exposes order JSON
         │  (e-commerce)   │
         └─────────────────┘

         ┌─────────────────┐     ┌───────────────────┐
         │  Luca & Claire  │────>│   Claude API       │
         │ (exhibit design)│     │  (Luca + Claire)   │
         └─────────────────┘     └───────────────────┘
```
