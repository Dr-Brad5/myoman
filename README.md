# Myoman

Interactive muscle and nerve study tools.

- `index.html`: Muscle & Nerve Explorer. Click a muscle group to see its nerve, roots and actions.
- `game.html`: Nerve Quiz. A 60-second round of naming the nerve and roots for highlighted muscles, with a shared scoreboard.
- `data.js`: the body diagram and all muscle facts, shared by both pages. Edit muscles or the figure here.
- `scoreboard-config.js`: connection settings for the online scoreboard.

## Setting up the shared scoreboard (Supabase, free)

1. Sign up at https://supabase.com and create a new project (any name and region).
2. In the project, open **SQL Editor**, paste the SQL below and click **Run**.
3. Open **Project Settings → API** (or **API Keys**). Copy the **Project URL** and the
   **anon public** key (newer projects call it the **publishable** key).
4. Paste both into `scoreboard-config.js` and commit.

```sql
create table public.scores (
  id bigint generated always as identity primary key,
  name text not null check (char_length(name) between 1 and 24),
  score int not null check (score >= 0),
  correct int not null check (correct >= 0),
  answered int not null check (answered between correct and 100),
  created_at timestamptz not null default now(),
  -- every correct answer is worth 1 to 5 points
  check (score between correct and correct * 5)
);

alter table public.scores enable row level security;

create policy "Anyone can read scores"
  on public.scores for select to anon using (true);

create policy "Anyone can add a score"
  on public.scores for insert to anon with check (true);

grant select, insert on public.scores to anon;
```

Visitors can only read scores and add new ones; they cannot edit or delete them.
To remove a score (for example a joke name), delete the row in Supabase's **Table Editor**.

If `scoreboard-config.js` is left empty, the quiz keeps a scoreboard in each player's own browser instead.
