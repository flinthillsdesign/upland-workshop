# Upland Workshop

The shared brain for the Upland Workshop app family. This is where we brainstorm, refine designs, document shared patterns, plan future apps, and maintain the big picture across the whole ecosystem.

This repo doesn't have a backend or UI — it's a working space. Patterns get designed here, then carried into individual app repos.

## The Family

| App | Domain | Stack | Repo |
|-----|--------|-------|------|
| **Upland Website** | uplandexhibits.com | Python (Plain), PostgreSQL, Heroku | `flinthillsdesign/upland` |
| **ODIN** | odin.uplandexhibits.com | TypeScript, Netlify, Turso | `flinthillsdesign/upland-odin` |
| **Inquiry Hub** | inquiries.uplandexhibits.com | TypeScript, Netlify, Turso | `flinthillsdesign/upland-inquiries` |
| **Scheduler** | schedules.uplandexhibits.com | TypeScript, Netlify, Turso | `flinthillsdesign/upland-scheduler` |
| **Luca & Claire** | TBD | TBD | TBD |

### Upland Website
The primary marketing website, e-commerce storefront, and order management system. Python/Plain on Heroku with PostgreSQL. Has its own auth system. Standalone — not on the shared auth or Netlify stack, and that's intentional.

### Satellite Apps (Netlify + Turso)
ODIN, Inquiry Hub, and Scheduler share a Turso auth database and JWT secret. They follow the same conventions: vanilla TypeScript, esbuild, single Netlify function with hand-rolled router, dual Turso databases (shared auth + app-specific), ensureSchema() migrations, Postmark email, Claude AI.

### Luca & Claire (planned)
The graphic designer of the family. Replaces the current static Previews app.
- **Luca** (Layout Utility Curatorial Assistant) — helps curators create exhibit layouts by placing content thoughtfully and drafting copy when needed
- **Claire** (Creative Layout AI Rendering Engine) — the visual designer, handles rendering and graphic design

## Key Files

- `ecosystem.md` — Full writeup of each app, shared patterns, and how they connect
- `brand/` — Shared visual identity, design explorations, and brand assets
  - `icon-preview-unified.html` — The definitive icon suite (all apps, unified dark background)
  - `icon-preview-micro.html` — Micro-sized icon variations
  - `ecosystem-diagram.html` — Visual architecture diagram
  - `icons/` — Individual app favicon SVGs (the source of truth)
  - `logos/` — Upland Exhibits wordmark, mascot, dark/white variants
  - `explorations/` — Earlier design iterations (ODIN exploration, three-set comparison)

## Conventions (Satellite Apps)

- Vanilla TypeScript (no React/Vue/Svelte), esbuild, oxlint, oxfmt
- Single Netlify function with hand-rolled router
- Dual Turso databases (shared auth + app-specific)
- `ensureSchema()` idempotent migrations
- Postmark for email, Claude API for AI features
- Instrument Sans font, #A5BC39 accent, dark gradient #161A22 to #0A0D12
