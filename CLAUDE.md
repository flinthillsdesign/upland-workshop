# Upland Workshop

The working space and brand home for the Upland app suite. No backend, no UI —
patterns and brand assets are designed here, then carried into the app repos.

## The family (current)

| App | Domain | Notes |
|-----|--------|-------|
| Website | uplandexhibits.com | Python/Heroku, own auth — intentionally standalone |
| ODIN | odin.uplandexhibits.com | operational hub; owns users + admin |
| Scheduler | schedules.uplandexhibits.com | reference satellite app |
| Inquiry Hub | inquiries.uplandexhibits.com | dormant — being rethought |
| Agreements | agreements.uplandexhibits.com | contracts + digital signatures |
| Strategy | strategy.uplandexhibits.com | cookie-gated strategy site |
| Budgets | budgets.uplandexhibits.com | Anthony's React app, joining incrementally |

Shared libraries: `@upland/auth` (upland-auth) and `@upland/shared` (upland-shared),
consumed as tag-pinned git deps. Retired and experimental apps (Claire, Quotes,
Tony, Previews, …) live in the `upland-apps-experimental` workspace folder; their
icons remain archived in `brand/icons/`.

## Brand assets

`brand/icons/` is the source of truth for every app favicon. Edit here, then copy
into the app repo — `public/favicon.svg` for satellites, `app/assets/icons/favicon.svg`
for the website, `client/public/favicon.svg` for budgets. Light variants
(`*-light.svg`) exist for light backgrounds.

Also in `brand/`: `logos/` (wordmark, mascot, dark/white variants),
`icon-preview.html` (the whole suite at every size), `explorations/` (active design
work, with earlier iterations in `explorations/archive/`).

Satellites auto-deploy on push; the website deploys manually via Heroku.
