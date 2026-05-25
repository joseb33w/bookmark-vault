# Goal

Single-page web app: Bookmark Vault. Email/password auth screen is the FIRST thing the user sees;
after sign-in they see a Save form (URL + Title + Save button) and a list of their bookmarks where
each row is a clickable link with a Delete button.

Visual layer: glassmorphism cards with cursor-tracked 3D tilt, animated mesh-gradient background
with floating colored orbs that drift in 3D, 3D flip-in entrance animations on bookmark rows,
3D-press button hover, animated brand mark badge, gradient-text headlines. `prefers-reduced-motion`
disables every animation/transform.

# Files to touch

- `index.html` — the entire app (vanilla HTML/CSS/JS, Supabase JS SDK from the supported CDN).
- `.env.example` — Supabase URL / publishable key / table-name pattern, for anyone who wants to fork.
- `README.md` — what it is, how to run locally, how to deploy.
- `PLAN.md` — this file.

No build step. No bundler. No framework. Single static file you can open with any static server.

# Backend

Supabase Postgres table `public."usr_nmexs7bytxq2_bookmarks"` with:
- `id uuid pk`, `user_id text not null` (FK conceptually to `auth.users.id`), `url text not null`,
  `title text not null`, `created_at timestamptz default now()`.
- RLS ON with one policy: `auth.uid()::text = user_id` for ALL operations (authenticated only).
- Explicit GRANTs to `authenticated` and `service_role` (`SELECT, INSERT, UPDATE, DELETE`).

Schema is applied directly to the shared Supabase project (already done before opening the original PR).
This design-pass round does NOT touch the schema.

# Verification approach

1. Pre-flight: visual code review (no TS/lint in a vanilla HTML repo to skip).
2. Backend: drive the REST API via the real UI in a headless browser, using a JWT from a
   freshly created (admin-confirmed) test user. Confirm Bearer-auth POSTs hit Supabase.
3. Frontend: serve the file locally, drive in headless Chromium via Playwright:
   - Auth card renders, cursor-tracked 3D tilt produces a `matrix3d` transform on hover.
   - All 4 background orbs present, transform changes between frames (animation is live).
   - Brand mark 3D float animation transform changes between frames.
   - Sign in via the real form, see main view.
   - Add two bookmarks via the real form, see both appear with entrance animation.
   - Hover a bookmark, confirm transform changes (3D lift via :hover).
   - Confirm `href` and `target=_blank` on each bookmark anchor.
   - Delete one with the 3D flip-out animation, see it gone from list AND from DB.
   - Sign out returns to auth screen.
4. Integration: in the headless run, watch the network panel for real requests to
   `xhhmxabftbyxrirvvihn.supabase.co` with `Authorization: Bearer <jwt>` headers.
5. Clean up the test user + all rows via the admin API before opening the PR.

# Out of scope

- Editing existing bookmarks (only create + delete; matches the brief).
- URL validation beyond `type="url"` on the input.
- Social sign-in / magic links / OAuth (brief says "simple email/password").
- Sharing bookmarks between users.
- Three.js / WebGL — the 3D motions are pure CSS perspective + transform + a small JS
  pointer-tracking helper. Keeps the single-file no-build constraint.

# Round 3 — Live search filter (interactive feature)

Small, frontend-only addition: a pill-shaped search input above the bookmark list that
filters items live as you type, with the matched substring highlighted in both title and
URL. Keyboard-driven: `/` focuses the input from anywhere on the main view, `Esc` clears
it. Result count flips to `(M of N)` while filtering, and an empty-search panel with a
magnifying-glass icon takes over when zero items match. No schema change. No SDK change.
Filter state survives saves and deletes (a new bookmark that matches the active query
joins the visible list; deleting a visible match leaves the filter intact). Animations
respect `prefers-reduced-motion`.
