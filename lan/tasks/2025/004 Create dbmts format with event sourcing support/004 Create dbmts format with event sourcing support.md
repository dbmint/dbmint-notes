---
status: done
---

# 1 Objective

The project lives in [gh dbmint/dbmints_rs](https://github.com/dbmint/dbmints_rs).

We achieved a proof of concept of historical tables while working on [gh  deltachives/2025-002-credit-store-demo-rs](https://github.com/deltachives/2025-002-credit-store-demo-rs). It can be found in the sql [here](https://github.com/deltatraced/delta-trace/blob/eb6048478ab68dfe00d89842b714e12c378f0755/lan/protos/2025/001%20Rust%20Diesel%20Event%20Sourcing/tasks/2025/000%20Implement%20the%20Event%20Accumulator/investigations/004%20Investigate%20options%20for%20materializing%20views%20into%20tables%20using%20SQL.md?plain=1#L366) from [004 Investigate options for materializing views into tables using SQL.md](https://github.com/deltatraced/delta-trace/blob/webview/lan/protos/2025/001%20Rust%20Diesel%20Event%20Sourcing/tasks/2025/000%20Implement%20the%20Event%20Accumulator/investigations/004%20Investigate%20options%20for%20materializing%20views%20into%20tables%20using%20SQL.md)

We need to be able to generate all that boilerplate from a single user-relevant table like `coin_store_diffs`

```sql
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  person TEXT NOT NULL,
  coins INTEGER NOT NULL
);
```

It must have an `obj_id`, or an error is raised, and the user gets to specify the type of accumulation:

```sql
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL
);
```

A `.dbmts` file is a superset of SQL, plus derives and configs like `#[...]`.

The above table then will generate the entirety of the remaining boilerplate views, triggers, tables in a `.sql` file. For user interest, they will be able to use `coin_store_hist`, `coin_store_hist_partial`, `coin_store_events`, `coin_store_events_grouped`, `coin_store_events_grouped_partial`.

Note that this is focused on `sqlite3` for now.

# 2 Journal

2025-10-09 Wk 41 Thu - 09:06 +03:00

Spawn [[000 Create rust template project]] ^spawn-task-49a28a

2025-10-09 Wk 41 Thu - 11:11 +03:00

Spawn [[001 Create basic parsing of derive and es for dbmts]] ^spawn-task-2f3be1

2025-10-09 Wk 41 Thu - 16:55 +03:00

Spawn [[002 Group settings by table and enforce some conditions on the table and settings for dbmts]] ^spawn-task-a09a05

2025-10-09 Wk 41 Thu - 21:16 +03:00

Spawn [[003 Impl writing the expanded event sourcing output given the grouped settings by table for dbmts]] ^spawn-task-8f580d

2025-10-10 Wk 41 Fri - 00:23 +03:00

OK!