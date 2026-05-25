# Bookmark Vault

A single-page app for saving and managing your bookmarks. Sign in, paste a URL and a title,
hit Save. Your bookmarks show up as a list of clickable links, each with a Delete button.
A pill-shaped search bar above the list filters your bookmarks live as you type, with
matched substrings highlighted in both the title and the URL.

The interface uses glassmorphic cards, a mesh-gradient background with floating animated
orbs, cursor-tracked 3D tilt on every card, a slowly rotating 3D brand mark, and 3D
flip-in/out entrance animations on bookmark rows. All motion respects
`prefers-reduced-motion: reduce`.

## Search shortcuts

- `/` — focus the search bar from anywhere on the main screen.
- `Esc` — clear the current query (or blur the input if already empty).
- Click the **×** inside the input to clear instantly.

The result count next to the section title shows either `(N)` when no search is active or
`(M of N)` when filtering. If a query has zero matches, a magnifying-glass empty state
takes over the list area.

No build step, no framework, no node_modules. Just one HTML file.

## Stack

- Single static `index.html` — vanilla HTML, CSS and ES-module JavaScript.
- [`@supabase/supabase-js`](https://github.com/supabase/supabase-js) loaded from
  [jsDelivr](https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/+esm) (Supabase officially
  supports this CDN path).
- Backend: Supabase Postgres + Supabase Auth (email/password). One table, one RLS policy,
  scoped to the signed-in user via `auth.uid()::text = user_id`.

## Run it locally

You only need a static file server:

```bash
# pick one
python3 -m http.server 8000
# or
npx serve -p 8000 .
```

Open <http://localhost:8000/> and sign up. Supabase sends a confirmation email; after you
click the link in it, come back and sign in.

## Configuration

The Supabase URL, publishable (anon) key, and table name live at the top of `index.html`'s
inline `<script type="module">`. See `.env.example` for the values.

Because this is a static HTML page (no bundler), changing those values means editing
`index.html` directly. If you fork this app for your own Supabase project:

1. Copy `.env.example` to `.env` so you remember your values.
2. Update `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `TABLE` at the top of the inline script
   in `index.html`.
3. Create a matching table in your Supabase project with RLS enabled — see the schema
   below.

## Database schema

```sql
create table public.bookmarks (
  id uuid primary key default gen_random_uuid(),
  user_id text not null,
  url text not null,
  title text not null,
  created_at timestamptz not null default now()
);

alter table public.bookmarks enable row level security;

create policy "auth_user_access" on public.bookmarks
  for all to authenticated
  using (auth.uid()::text = user_id)
  with check (auth.uid()::text = user_id);

grant select, insert, update, delete on public.bookmarks to authenticated;
```

The deployed instance uses a per-user-prefixed name (`usr_<id>_bookmarks`) because the
Supabase project is shared across users; the schema itself is otherwise identical.

## License

MIT — see [`LICENSE`](./LICENSE).
