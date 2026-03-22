# Luca & Claire — App Specification

## Identity

| | |
|---|---|
| **App name** | Luca & Claire |
| **Domain** | luca.uplandexhibits.com |
| **Port** | 3003 (local dev) |
| **Repo** | `flinthillsdesign/upland-luca-claire` |
| **Stack** | TypeScript, Netlify Functions, Turso |
| **Replaces** | Upland Previews (`flinthillsdesign/upland-previews`) |

Follows the satellite app conventions: vanilla TypeScript, esbuild, oxlint/oxfmt, single Netlify function with hand-rolled router, dual Turso databases, shared JWT auth.

Uses ODIN's refined dual-database pattern with separate `auth-storage.ts` and `storage.ts` modules.

---

## What It Does

Luca & Claire is an AI-assisted exhibit panel design tool. It replaces the static Previews app with an interactive environment where curators and designers create exhibit panels, get AI help with content and visual design, and share previews with clients.

**Luca** (Layout Utility Curatorial Assistant) helps with content — what goes on the panel, narrative flow across the exhibit, copy drafting and editing, curatorial guidance.

**Claire** (Creative Layout AI Rendering Engine) helps with design — typography, color palettes, layout composition, visual refinement, preparing panels for client review.

They're two AI team members, each powered by Claude, with overlapping skills but distinct specialties. Like real colleagues — either can help with most things, but each brings their own perspective.

### Core Capabilities

1. **Projects** — Create and manage exhibit projects. Each project has a client, a set of panels, and style defaults (fonts, colors, voice).

2. **Panels** — Define panels with physical dimensions, panel type, mounting height. The panel is the unit of design — a single exhibit graphic that goes on a wall or in a Montera frame.

3. **Content Editing** — Block in text, images, and captions on panels using an HTML/CSS-based editor. Panels render as styled HTML, just like the current Previews app. The content is structured (title, body, caption, etc.) but visually rendered.

4. **AI Assistance** — Chat with Luca about content or Claire about design. They see the current panel state and project context. They can suggest copy, recommend layouts, critique compositions, draft alternative text, propose color palettes.

5. **Gallery / Preview** — Client-facing gallery view inherited from Previews. Thumbnail grid, detail viewer with navigation, filter by panel type, CVZ zone guides overlay. Accessed via shareable token links.

6. **Export** — PNG export via html2canvas (proven in Previews). Future: Illustrator file generation (workflow TBD, may involve claude-the-designer's exhibit-design skill).

---

## The Two Personas

Luca and Claire are implemented as two system prompts that share the same Claude API integration. Each has domain expertise embedded from the exhibit-design skill specs.

### Luca — Content & Curation

System prompt emphasis:
- Exhibit narrative flow — how panels tell a story through the gallery
- Content hierarchy — what's the title, what's the lede, what's the body
- Copy drafting — writing body text, captions, callouts from source material
- Word count discipline — 150 words max for casual visitors, 250 absolute max
- Curatorial voice — matching the client's tone (academic, playful, reverent, etc.)
- Content zones — what content belongs in the CVZ vs. title zone vs. secondary zone

### Claire — Design & Composition

System prompt emphasis:
- Typography system — role-based sizing, leading ratios, font pairing
- Color palettes — accent colors, background treatments, contrast ratios
- Layout composition — margins, columns, image/text balance, white space
- Visual hierarchy — how the eye moves through the panel
- Panel type conventions — what each panel type looks like and why
- CVZ compliance — readable text within 36"–67" from floor

### How They Work Together

In the UI, there's a chat/assistant panel. The user can address either persona:
- "Luca, draft body text for this panel about sea turtle migration"
- "Claire, this feels too text-heavy. What if we added an image?"
- "What do you two think about the flow from panels 3 to 5?"

The AI routes to the appropriate system prompt based on who's addressed, or uses context to pick the right one. When both are asked, both perspectives are incorporated.

---

## Exhibit Design Knowledge

This domain knowledge is embedded into AI system prompts and used for validation/guidance in the UI. Source of truth: `claude-the-designer/.claude/skills/exhibit-design/`.

### Panel Types

| Type | Description | Default Size | Mount Height |
|------|-------------|-------------|-------------|
| `title_panel` | Exhibition title — hero image, big title | 36" x 48" | 24" |
| `interpretive` | Main content — text, images, captions | 24" x 36" | 30" |
| `reader_rail` | Low horizontal, paired with objects | 48" x 15" | 30" |
| `object_label` | Small artifact labels | 8" x 4" | 42" |
| `section_intro` | Section introduction panel | 24" x 78" | 6" |

Each type has variants (e.g., interpretive has vertical, horizontal, large, montera_standard, montera_wide).

### Comfortable Viewing Zone (CVZ)

| Height | Landmark |
|--------|----------|
| 67" | CVZ top |
| 60" | Primary reading center |
| 48" | ADA max forward reach |
| 42" | Wheelchair eye level |
| 36" | CVZ bottom |

### Content Zones

| Zone | Floor Height | Allowed Content |
|------|-------------|-----------------|
| Title Zone | 67"–84" | Title, subtitle, hero images, decorative |
| Primary (CVZ) | 36"–67" | Body text, headings, images, captions, callouts |
| Secondary | 12"–36" | Credits and logos only |
| Dead Zone | 0"–12" | Nothing |

### Typography Roles

| Role | Default Size | Leading | Min Size |
|------|-------------|---------|----------|
| title | scales with panel | 1.1x | 72pt |
| subtitle | 72pt | 1.2x | 48pt |
| section_head | 48pt | 58pt | 36pt |
| lede | 40pt | 54pt | 36pt |
| body | 36pt | 50pt | 36pt |
| caption | 24pt | 34pt | 24pt |
| credit | 18pt | 25pt | 18pt |
| callout | 60pt | 72pt | 48pt |

---

## Data Model

### `projects`

| Column | Type | Notes |
|--------|------|-------|
| id | TEXT PK | nanoid |
| name | TEXT NOT NULL | Exhibition name |
| client | TEXT | Client/museum name |
| status | TEXT | draft, active, archived |
| style_defaults | TEXT | JSON — fonts, colors, voice/tone |
| created_at | TEXT | ISO timestamp |
| updated_at | TEXT | ISO timestamp |

### `panels`

| Column | Type | Notes |
|--------|------|-------|
| id | TEXT PK | nanoid |
| project_id | TEXT NOT NULL | FK to projects |
| title | TEXT NOT NULL | Panel title |
| type | TEXT NOT NULL | title_panel, interpretive, reader_rail, object_label, section_intro |
| variant | TEXT | Type variant (e.g., montera_standard) |
| code | TEXT | Internal reference (e.g., "1.1") |
| width_in | REAL NOT NULL | Physical width in inches |
| height_in | REAL NOT NULL | Physical height in inches |
| mount_height_in | REAL | Inches from floor to panel bottom |
| sort_order | INTEGER | Display order within project |
| content | TEXT | JSON — structured panel content (see below) |
| style_overrides | TEXT | JSON — panel-specific style overrides |
| created_at | TEXT | ISO timestamp |
| updated_at | TEXT | ISO timestamp |

#### Panel Content Structure (JSON)

The `content` field stores the panel's structured content as JSON. Each content block has a role, position, and text/image data:

```json
{
  "blocks": [
    {
      "id": "b1",
      "role": "title",
      "text": "Sea Turtle Migration",
      "style": {}
    },
    {
      "id": "b2",
      "role": "body",
      "text": "Every year, loggerhead sea turtles...",
      "style": {}
    },
    {
      "id": "b3",
      "role": "image",
      "src": "/uploads/turtle-map.jpg",
      "alt": "Migration route map",
      "style": {}
    },
    {
      "id": "b4",
      "role": "caption",
      "text": "Migration routes of the Atlantic loggerhead",
      "style": {}
    }
  ],
  "background": {
    "color": "#1a2332",
    "gradient": null,
    "image": null
  },
  "css": ""
}
```

The `css` field allows freeform CSS for advanced styling — this is how the current Previews panels work (hand-coded HTML/CSS), and it provides an escape hatch for designs that don't fit neatly into the block model.

### `conversations`

| Column | Type | Notes |
|--------|------|-------|
| id | TEXT PK | nanoid |
| project_id | TEXT NOT NULL | FK to projects |
| panel_id | TEXT | FK to panels (null = project-level conversation) |
| persona | TEXT NOT NULL | luca, claire |
| messages | TEXT NOT NULL | JSON array of {role, content, timestamp} |
| created_at | TEXT | ISO timestamp |

### `settings`

| Column | Type | Notes |
|--------|------|-------|
| id | INTEGER PK | Always 1 |
| data | TEXT NOT NULL | JSON singleton |

---

## API Surface

### Auth (public)
- `POST /api/login` — username + password → JWT
- `POST /api/forgot-password` — send reset email
- `POST /api/reset-password` — token + new password

### Projects (authenticated)
- `GET /api/projects` — list all projects
- `POST /api/projects` — create project
- `GET /api/projects/:id` — get project with panel list
- `PUT /api/projects/:id` — update project
- `DELETE /api/projects/:id` — archive/delete project

### Panels (authenticated)
- `GET /api/projects/:id/panels` — list panels for project
- `POST /api/projects/:id/panels` — create panel
- `GET /api/panels/:id` — get panel with content
- `PUT /api/panels/:id` — update panel (content, metadata, style)
- `DELETE /api/panels/:id` — delete panel
- `PUT /api/panels/reorder` — reorder panels within project

### AI (authenticated)
- `POST /api/panels/:id/ask-luca` — send message to Luca about this panel
- `POST /api/panels/:id/ask-claire` — send message to Claire about this panel
- `POST /api/projects/:id/ask-luca` — project-level Luca conversation
- `POST /api/projects/:id/ask-claire` — project-level Claire conversation
- `GET /api/conversations/:id` — get conversation history

### Gallery (token-authenticated)
- `GET /api/projects/:id/gallery` — get gallery data for client preview
- `POST /api/projects/:id/share` — generate shareable token link

### Users (superadmin only)
- `GET /api/users` — list users
- `POST /api/users` — create user
- `PUT /api/users/:id` — update user
- `DELETE /api/users/:id` — delete user

---

## Frontend Structure

### Pages

- `index.html` — Login
- `dashboard.html` — Project list
- `project.html` — Project view (panel grid, project settings)
- `editor.html` — Panel editor (the main workspace — panel preview + content editing + AI chat)
- `gallery.html` — Client-facing gallery (shareable, token-auth)
- `settings.html` — App settings, user management
- `forgot.html` / `reset.html` — Password reset flow

### The Editor (editor.html)

The core of the app. Three regions:

1. **Panel Canvas** (center) — Live HTML/CSS rendering of the panel at scale. Shows CVZ zone guides when toggled. Content blocks are selectable and editable.

2. **Content Panel** (left or collapsible) — Structured list of content blocks (title, body, images, captions). Add/remove/reorder blocks. Edit text inline or in a focused editor. Set roles and styles.

3. **AI Chat** (right or collapsible) — Conversation with Luca or Claire. Shows context (current panel, project). Responses can include suggested content that can be applied with one click.

### The Gallery (gallery.html)

Inherited from Previews:
- Thumbnail grid with panel cards
- Filter by panel type
- Detail overlay with prev/next navigation
- CVZ zone guides toggle
- PNG export
- Keyboard navigation (arrows, escape)
- Summary bar with panel count and type breakdown

---

## Integrations

| System | Method | Purpose |
|--------|--------|---------|
| Turso Auth DB | `TURSO_AUTH_URL` | Shared user authentication |
| Turso App DB | `TURSO_URL` | App-specific data (projects, panels, conversations) |
| Claude API | `CLAUDE_API_KEY` | Luca and Claire AI assistance |
| Postmark | `POSTMARK_API_TOKEN` | Password reset emails, gallery share notifications |

No ODIN/Scheduler/Inquiry Hub integration initially. ODIN may eventually read from Luca & Claire for project status context.

---

## Environment Variables

```bash
# Auth (shared across satellite apps)
JWT_SECRET=<shared-secret>
TURSO_AUTH_URL=file:./data/auth.db
TURSO_AUTH_TOKEN=<token>

# App database
TURSO_URL=file:./data/local.db
TURSO_TOKEN=<token>

# AI
CLAUDE_API_KEY=<anthropic-key>

# Email
POSTMARK_API_TOKEN=<token>
POSTMARK_FROM_EMAIL=info@uplandexhibits.com

# Local dev
PORT=3003
```

---

## Directory Structure

```
upland-luca-claire/
├── netlify/functions/
│   └── api.ts                 # Single function, hand-rolled router
├── lib/
│   ├── auth.ts                # JWT + bcrypt utilities
│   ├── auth-storage.ts        # Shared auth DB (TURSO_AUTH_URL)
│   ├── storage.ts             # App DB (TURSO_URL) — projects, panels, conversations
│   ├── ai.ts                  # Claude API — Luca and Claire system prompts
│   └── email.ts               # Postmark
├── public/
│   ├── index.html             # Login
│   ├── dashboard.html         # Project list
│   ├── project.html           # Project view
│   ├── editor.html            # Panel editor (main workspace)
│   ├── gallery.html           # Client-facing gallery
│   ├── settings.html          # App settings
│   ├── forgot.html            # Password reset request
│   ├── reset.html             # Password reset form
│   ├── js/
│   │   ├── api.ts             # API client
│   │   ├── login.ts
│   │   ├── dashboard.ts
│   │   ├── project.ts
│   │   ├── editor.ts          # Panel editor logic
│   │   ├── gallery.ts         # Gallery viewer (inherited from Previews)
│   │   └── settings.ts
│   └── css/
│       ├── shared.css         # Global styles
│       ├── editor.css         # Editor workspace
│       └── gallery.css        # Gallery styles
├── data/                      # Local SQLite databases
│   ├── auth.db
│   └── local.db
├── server.ts                  # Express dev server
├── build.js                   # esbuild config
├── bootstrap.js               # Seed initial users
├── package.json
├── tsconfig.json
├── netlify.toml
├── .env
├── .oxlintrc.json
└── .gitignore
```

---

## What's Intentionally Deferred

- **Illustrator export pipeline** — The path from HTML panels to production .ai files is TBD. May involve claude-the-designer's exhibit-design skill, or a dedicated export feature, or something else. Will be figured out as the app takes shape.
- **Image upload/storage** — Initially, images can be referenced by URL. A proper upload system (probably Netlify Blobs or an S3-compatible store) comes later.
- **Real-time collaboration** — Single-user editing for now.
- **Version history** — Panel content versioning/undo is deferred.
- **ODIN integration** — ODIN reading from Luca & Claire for project context comes after the app is stable.
