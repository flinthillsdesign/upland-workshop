# Upland Workshop

The working space and brand home for the Upland app suite. No backend, no UI —
patterns and brand assets are designed here, then carried into the app repos.

## The family (current)

| App | Domain | Notes |
|-----|--------|-------|
| Website | uplandexhibits.com | Python/Heroku, own auth — intentionally standalone |
| ODIN | odin.uplandexhibits.com | operational hub; owns users + admin |
| Scheduler | schedules.uplandexhibits.com | reference satellite app |
| Agreements | agreements.uplandexhibits.com | contracts + digital signatures |
| Strategy | strategy.uplandexhibits.com | cookie-gated strategy site |
| Budgets | budgets.uplandexhibits.com | Anthony's React app, joining incrementally |
| Proposals | proposals.uplandexhibits.com | client proposal sharing — short link + access code |
| Graphics | graphics.uplandexhibits.com | object labels: write, proof, order; fly.io, not Netlify |

Shared libraries: `@upland/auth` (upland-auth) and `@upland/shared` (upland-shared),
consumed as tag-pinned git deps. Retired and experimental apps live in the
`upland-apps-experimental` workspace folder — the folder is the list, and each
repo's `CLAUDE.md` opens with its status. Their icons remain archived in
`brand/icons/`.

## Brand assets

`brand/icons/` is the source of truth for every app favicon. Edit here, then copy
into the app repo — `public/favicon.svg` for satellites, `app/assets/icons/favicon.svg`
for the website, `client/public/favicon.svg` for budgets. Light variants
(`*-light.svg`) exist for light backgrounds.

Also in `brand/`: `logos/` (wordmark, mascot, dark/white variants),
`icon-preview.html` (the whole suite at every size), `explorations/` (active design
work, with earlier iterations in `explorations/archive/`).

Satellites auto-deploy on push; the website deploys manually via Heroku.
