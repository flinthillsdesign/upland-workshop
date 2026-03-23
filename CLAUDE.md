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
| **Claire** | claire.uplandexhibits.com | TypeScript, Netlify, Turso | `flinthillsdesign/upland-claire` |

### Upland Website
The primary marketing website, e-commerce storefront, and order management system. Python/Plain on Heroku with PostgreSQL. Has its own auth system. Standalone — not on the shared auth or Netlify stack, and that's intentional.

### Satellite Apps (Netlify + Turso)
ODIN, Inquiry Hub, Scheduler, and Claire share a Turso auth database and JWT secret. They follow the same conventions: vanilla TypeScript, esbuild, single Netlify function with hand-rolled router, dual Turso databases (shared auth + app-specific), ensureSchema() migrations, Postmark email, Claude AI.

### Claire
The graphic designer of the family. Replaces the static Previews app with an AI-assisted exhibit panel design tool. Same satellite stack (Netlify + Turso + shared auth). Port 3003.
- **Claire** (Creative Layout AI Rendering Engine) — AI design assistant handling content, layout, typography, color, and visual refinement
- See `claire/spec.md` for the full app specification

## Key Files

- `ecosystem.md` — Full writeup of each app, shared patterns, and how they connect
- `brand/` — Shared visual identity, design explorations, and brand assets
  - `icon-preview.html` — The definitive icon suite (all apps at 96px, 52px dock, 32px micro, 16px tiny)
  - `ecosystem-diagram.html` — Visual architecture diagram (needs updating)
  - `icons/` — Individual app favicon SVGs (the source of truth)
  - `logos/` — Upland Exhibits wordmark, mascot, dark/white variants
  - `explorations/` — Current design work
  - `explorations/archive/` — Earlier design iterations

## Brand Asset Distribution

This repo is the source of truth for all app icons. Each app repo copies its favicon from here. Don't modify favicons in app repos directly — update them here first, then copy over.

| App | Source (this repo) | Destination (app repo) |
|-----|-------------------|----------------------|
| Website | `brand/icons/website.svg` | `app/assets/icons/favicon.svg` |
| ODIN | `brand/icons/odin.svg` | `public/favicon.svg` |
| Inquiry Hub | `brand/icons/inquiry-hub.svg` | `public/favicon.svg` |
| Scheduler | `brand/icons/scheduler.svg` | `public/favicon.svg` |
| Claire | `brand/icons/claire.svg` | `public/favicon.svg` |

Light variants (`*-light.svg`) are available for each app for use on light backgrounds.

**Deploying updates to app repos:**
- **Satellite apps** (ODIN, Inquiry Hub, Scheduler, Claire): commit and push — Netlify auto-deploys
- **Upland Website**: commit, push, then deploy manually via `git push heroku master` (or Heroku dashboard)

## Conventions (Satellite Apps)

- Vanilla TypeScript (no React/Vue/Svelte), esbuild, oxlint, oxfmt
- Single Netlify function with hand-rolled router
- Dual Turso databases (shared auth + app-specific)
- `ensureSchema()` idempotent migrations
- Postmark for email, Claude API for AI features
- Instrument Sans font, #A5BC39 accent, dark gradient #161A22 to #0A0D12
