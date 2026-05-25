# Goal

Single-page web app: Bookmark Vault. Email/password auth screen is the FIRST thing the user sees;
after sign-in they see a Save form (URL + Title + Save button) and a list of their bookmarks where
each row is a clickable link with a Delete button.

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

Schema is applied directly to the shared Supabase project (already done before opening the PR).

# Verification approach

1. Pre-flight: visual code review (no TS/lint in a vanilla HTML repo to skip).
2. Backend: drive the REST API with `curl` using a JWT from a freshly created (admin-confirmed) test user.
   - Positive: insert + select + delete as that user — all succeed.
   - Negative: same operations as a different user — RLS blocks them.
3. Frontend: serve the file locally, drive in headless Chromium via Playwright:
   - Sign-up shows the "check your email" state for an unconfirmed signup.
   - Pre-confirm a verify-user via the admin API, then sign in via the real UI form.
   - Add two bookmarks via the real form, see both appear in the list.
   - Confirm the link points where it should (`href` attribute) and is clickable.
   - Delete one, see it disappear from the list and from the database.
   - Sign out returns to auth screen.
4. Integration: in the headless run, watch the network panel for real requests to
   `xhhmxabftbyxrirvvihn.supabase.co` with `Authorization: Bearer <jwt>`.
5. Clean up all test users and rows via the admin API before opening the PR.

# Out of scope

- Editing existing bookmarks (only create + delete; matches the brief).
- URL validation beyond `type="url"` on the input.
- Social sign-in / magic links / OAuth (brief says "simple email/password").
- Sharing bookmarks between users.
- Sorting/filtering/search on the list.
