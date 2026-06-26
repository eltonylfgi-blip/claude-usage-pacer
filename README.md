# Claude Usage Pacer

A tiny, single-file web app that helps you **spread your weekly Claude usage** so you don't burn
through it before the limit resets. You type in the usage % you see in the Claude app, and it tells
you **how far you can go today** to stay on an even pace — plus a sustainable per-day rate and days
until reset.

No build step, no framework, no account required. It's one `index.html`. Open it and you're done.

> Why this exists: Claude's subscription plans have weekly usage limits. It's easy to spend most of
> your week's budget in the first two days and then get stuck. This gives you a simple "you can go up
> to X% today" number so you pace yourself.

## Features

- **"Today you can go up to X%"** — the headline number, the absolute weekly % you can reach today.
- Sustainable **per-day rate** and **days until reset**.
- A green / amber / red banner: *plenty of margin* · *on pace* · *going too fast, slow down*.
- A daily log of the current week with a little bar chart.
- The weekly cycle (7 days) **rolls over by itself**.
- **Works fully offline** — data is saved in your browser (localStorage).
- **Optional sync across devices** (PC ↔ phone) with your own free Supabase project.

## Run it

Pick whichever is easiest:

- **Just open it:** double-click `index.html`. That's it (local mode).
- **Put it online for free:** drag the folder onto [Vercel](https://vercel.com) or
  [Netlify](https://app.netlify.com/drop) (drag & drop, no GitHub needed). You get a URL you can
  open on your phone.

## Optional: sync across devices

By default everything is stored in your browser, so your phone and your laptop keep separate logs.
To sync them, give the app a free [Supabase](https://supabase.com) project as a backend.

1. Create a free project at supabase.com.
2. In the SQL editor, run:

   ```sql
   create table public.pacer_uso (
     owner text not null default 'me',
     fecha date not null,
     pct numeric not null,
     sonnet numeric,
     updated_at timestamptz not null default now(),
     primary key (owner, fecha)
   );
   alter table public.pacer_uso enable row level security;
   create policy "pacer_uso_anon_all" on public.pacer_uso for all to anon using (true) with check (true);
   grant select, insert, update, delete on public.pacer_uso to anon;

   create table public.pacer_config (
     owner text primary key default 'me',
     reset_date date not null default '2026-07-03',
     cycle_days int not null default 7,
     updated_at timestamptz not null default now()
   );
   alter table public.pacer_config enable row level security;
   create policy "pacer_config_anon_all" on public.pacer_config for all to anon using (true) with check (true);
   grant select, insert, update, delete on public.pacer_config to anon;
   ```

3. In `index.html`, fill in the CONFIG block near the top of the `<script>`:

   ```js
   const SUPABASE_URL = 'https://YOUR-PROJECT.supabase.co';
   const SUPABASE_KEY = 'sb_publishable_...'; // your publishable / anon key
   const OWNER        = 'me';
   ```

4. Deploy. Open it on any device — same data everywhere.

### A note on the key (read this)

The publishable / anon key is **meant to be public** — it ships inside the page, anyone can read it.
What protects your data is **Row Level Security (RLS)**, not hiding the key. The policy above is
*open* (anyone with the URL can read/write), which is fine for low-sensitivity data like your own
usage %. If you want it truly private (only you can write), add
[Supabase Auth](https://supabase.com/docs/guides/auth) and scope the RLS policies per user.

Also: if you push your own copy to a **public** GitHub repo with real keys filled in, those keys get
indexed. Keep keys in an untracked file, use a private repo, or just deploy by drag & drop.

## How the pacing math works

Given the reset date, today's date and your current used %:

- **Today's cap** = `(fraction of the week elapsed by end of today) × 100` — the % you'd be at if you
  spent the week perfectly evenly.
- **Left for today** = `cap − used` (clamped at 0).
- **Sustainable rate** = `(100 − used) / days remaining` — spread the rest evenly to finish right at
  the reset.

You type the % manually because there is currently **no public API** for your Claude plan usage; you
read it from the app's usage panel and enter it here.

## Contributing

The UI is currently in Spanish. **Translations (i18n) are very welcome** — open a PR. Bug reports and
ideas too.

## License

[MIT](LICENSE) © 2026 Tony (@eltonylfgi-blip)
