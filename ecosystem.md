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

### Claire (claire.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-claire`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3003 (local)

AI-assisted exhibit panel design tool. Replaces the static Previews app (`flinthillsdesign/upland-previews`) with an interactive environment where curators and designers create exhibit panels, get AI help with content and visual design, and share previews with clients.

**Claire** — Creative Layout AI Rendering Engine. A single AI assistant powered by Claude with exhibit design expertise. Handles the full spectrum: content strategy, narrative flow, copy drafting, typography, color palettes, layout composition, visual refinement, and preparing panels for client review.

**Key role in ecosystem:** Turns curatorial content into designed exhibit panels. Projects contain panels defined by physical dimensions and panel type. Content is structured (title, body, caption, images) and rendered as HTML/CSS. Clients view galleries via shareable token links.

**Integrations:**
- Claude API (Claire AI assistance)
- Postmark (password reset, gallery share notifications)

**Auth:** Shared Turso auth DB + JWT

---

### Quotes (quotes.uplandexhibits.com)
**Repo:** `flinthillsdesign/upland-quotes`
**Stack:** TypeScript, Netlify Functions, Turso
**Port:** 3004 (local)

AI-first quoting tool. Describe a project in plain English, and the AI generates a complete structured quote with sections, line items, pricing, and terms — drawing from a knowledge base of past quotes. Fully editable after generation. Shareable client links with view tracking. Print-ready via CSS.

**Key role in ecosystem:** Turns project scopes into professional quotes. AI-first generation from prompts, informed by historical pricing data. Web-first delivery — the client link IS the quote. Print stylesheet for paper output.

**Integrations:**
- Claude API (quote generation and conversational AI)
- Postmark (password reset, quote share notifications)

**Auth:** Shared Turso auth DB + JWT

---

## Standard App Order

When listing apps in UI (permission grids, nav menus, icon rows), always use this order:

1. **Storefront** (`storefront`) — Upland Website / e-commerce
2. **Inquiries** (`inquiries`) — Inquiry Hub
3. **Schedules** (`scheduler`) — Project Scheduler
4. **Quotes** (`quotes`) — AI Quoting
5. **Agreements** (`agreements`) — Contract Agreements
6. **ODIN** (`odin`) — Operations Dashboard
7. **Claire** (`claire`) — Exhibit Panel Design

The parenthesized values are the `app` identifiers used in `user_app_access`.

## Shared Infrastructure

### Auth Database (Turso)
A single Turso database (`upland-auth`) shared by all satellite apps.

**`users` table:**
| Field | Purpose |
|-------|---------|
| id | nanoid |
| username | Login identifier |
| password_hash | bcrypt |
| role | superadmin, manager, editor |
| name | Display name |
| email | For password reset |
| token_invalid_before | ISO timestamp — JWTs issued before this are rejected |

**`user_app_access` table:**
| Field | Purpose |
|-------|---------|
| user_id | FK to users.id |
| app | App identifier (`odin`, `scheduler`, `inquiries`, `claire`, `quotes`, etc.) |
| role | Per-app role: manager, editor, viewer |
| permissions | JSON, nullable — app-specific flags (e.g., `{"can_finance": true}` for Scheduler) |
| created_at | ISO timestamp |

**How it works:**
- **Superadmins** (users.role = `superadmin`) bypass app access checks — they can access every app with full permissions.
- **Everyone else** needs a row in `user_app_access` for each app they can use. No row = no access.
- **ODIN owns user management.** It's the only app with UI for creating users, granting app access, and setting per-app roles. All other apps read the auth DB but never write to it (except for password reset).
- **Per-app roles** (manager/editor/viewer) are interpreted by each app. The shared table stores them; each app decides what they mean.
- **App-specific permissions** go in the `permissions` JSON column. This replaces one-off columns like `can_finance` on the users table.

All satellite apps share a `JWT_SECRET`. A token minted by any app is valid in all others.

The Upland Website has its own auth and is not part of this system.

### Dual Database Pattern
Each satellite app connects to TWO Turso databases:
1. **Auth DB** (`TURSO_AUTH_URL`) — shared users + user_app_access tables (read-only except ODIN)
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

### Client Share Links

Several apps need to share resources with unauthenticated external users (clients reviewing quotes, viewing timelines, approving agreements, etc.). Each app implements this in its own app DB — the tokens are tightly coupled to app-specific resources — but follows the same pattern:

**Table: `share_tokens`** (or similar, named per app)
| Field | Purpose |
|-------|---------|
| id | nanoid |
| resource_type | What's being shared (e.g., `quote`, `gallery`, `agreement`) |
| resource_id | FK to the shared resource |
| token | Unique URL-safe string (word-based like `cedar-river-42` or nanoid) |
| created_by | User who created the link |
| created_at | ISO timestamp |
| expires_at | ISO timestamp, nullable (null = no expiry) |
| viewed_at | ISO timestamp, nullable — set on first view |
| revoked | Boolean, default false |

**Public route:** `GET /share/:token` — renders a read-only (or limited-action) view of the resource. No login required.

**Token format:** `adjective-noun-number` (e.g., `grumpy-moose-42`). One random adjective, one random noun, and a random number 10–99. Friendly, funny, and easy to read aloud. The canonical word lists:

**Adjectives (50):**
```
tiny, lanky, lumpy, fuzzy, wobbly, stubby, burly, plump, scruffy, pointy,
grumpy, sneaky, clever, bold, lazy, jolly, rowdy, shy, nosy, bossy,
clumsy, peppy, witty, cranky, giddy, sassy, spunky, zany, feisty, mellow,
rusty, dusty, mossy, crispy, smoky, frosty, muddy, misty, breezy, shady,
swift, lost, wild, odd, brisk, snug, dapper, plucky, wily, stout
```

**Nouns (50):**
```
sun, moon, cloud, frost, ember, mist, lake, river, brook, cove,
pine, oak, cedar, fern, moss, thistle, bramble, willow, briar, clover,
hawk, bear, fox, deer, wolf, crane, wren, moose, otter, badger,
raven, heron, bison, lynx, loon, newt, pika, yak,
amber, jade, pearl, copper, flint, cobalt, slate,
ridge, canyon, grove, meadow, bluff, hollow, fjord
```

**URL pattern:** `[app].uplandexhibits.com/share/[token]` — renders a stripped-down public view with no login required.

**Rules:**
- Check `revoked` and `expires_at` before rendering
- Set `viewed_at` on first access (tracks whether the client has seen it)

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
         │     Claire      │────>│   Claude API       │
         │ (exhibit design)│     │    (Claire)        │
         └─────────────────┘     └───────────────────┘

         ┌─────────────────┐     ┌───────────────────┐
         │     Quotes      │────>│   Claude API       │
         │  (AI quoting)   │     │  (quote generation)│
         └─────────────────┘     └───────────────────┘
```
