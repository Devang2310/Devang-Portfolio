# Devang Patel – Portfolio

A single-page portfolio (`index.html`) hosted on Vercel. Recommendations posted through the form are saved in a free Supabase database and appear on the site immediately.

## One-time setup (about 10 minutes)

### 1. Create the database
1. Sign up at https://supabase.com and create a new project (the free plan is fine). Pick a region in Canada or the US East.
2. In the project, open **SQL Editor → New query**, paste everything below and click **Run**.

```sql
create table public.recommendations (
  id           uuid primary key default gen_random_uuid(),
  created_at   timestamptz not null default now(),
  name         text not null check (char_length(name) between 2 and 80),
  email        text not null check (char_length(email) <= 120 and email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'),
  role         text check (char_length(role) <= 120),
  relationship text not null check (char_length(relationship) <= 60),
  body         text not null check (char_length(body) between 20 and 1500),
  approved     boolean not null default true
);

alter table public.recommendations enable row level security;

-- Visitors may post a recommendation and read approved ones.
create policy "visitors can post"
  on public.recommendations for insert to anon with check (true);
create policy "visitors can read approved"
  on public.recommendations for select to anon using (approved);

-- Visitors can never read emails or change the approved flag.
revoke all on public.recommendations from anon, authenticated;
grant insert (name, email, role, relationship, body) on public.recommendations to anon;
grant select (id, created_at, name, role, relationship, body) on public.recommendations to anon;
```

### 2. Connect the site
1. In Supabase, open **Project Settings → API** (or **API Keys**).
2. Copy the **Project URL** and the **publishable** key (older projects call it the **anon public** key). Never use the `service_role` or secret key.
3. In `index.html`, find these two lines and paste the values between the quotes:

```js
var SUPABASE_URL="https://your-project.supabase.co";
var SUPABASE_KEY="sb_publishable_...";
```

4. Commit and push. Vercel redeploys automatically.

Test it: open your site, post a test recommendation, and confirm it appears. Then delete the test row in Supabase.

## Day to day

- **See who posted (including their email):** Supabase → **Table Editor → recommendations**. Only you can see emails there.
- **Hide a recommendation:** set its `approved` value to `false`. Or delete the row.
- **The date** shown on each card is the day it was posted, set automatically.

## Good to know
- Free Supabase projects pause after about a week with no activity. If the form stops working, open the Supabase dashboard and click **Restore project**. The two LinkedIn recommendations are built into the page, so they always show.
- Because anyone with the link can post, check the table now and then and hide anything that isn't genuine.
