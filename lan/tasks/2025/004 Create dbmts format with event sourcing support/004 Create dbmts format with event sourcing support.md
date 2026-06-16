---
status: done
---

# 1 Objective

The project lives in [gh dbmint/dbmints_rs](https://github.com/dbmint/dbmints_rs).

We achieved a proof of concept of historical tables while working on [gh  deltachives/2025-002-credit-store-demo-rs](https://github.com/deltachives/2025-002-credit-store-demo-rs). It can be found in the sql [here](https://github.com/deltatraced/delta-trace/blob/eb6048478ab68dfe00d89842b714e12c378f0755/lan/protos/2025/001%20Rust%20Diesel%20Event%20Sourcing/tasks/2025/000%20Implement%20the%20Event%20Accumulator/investigations/004%20Investigate%20options%20for%20materializing%20views%20into%20tables%20using%20SQL.md?plain=1#L366) from [004 Investigate options for materializing views into tables using SQL.md](https://github.com/deltatraced/delta-trace/blob/webview/lan/protos/2025/001%20Rust%20Diesel%20Event%20Sourcing/tasks/2025/000%20Implement%20the%20Event%20Accumulator/investigations/004%20Investigate%20options%20for%20materializing%20views%20into%20tables%20using%20SQL.md)

We need to be able to generate all that boilerplate from a single user-relevant table like `coin_store_diffs`

````sql
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  person TEXT NOT NULL,
  coins INTEGER NOT NULL
);
````

It must have an `obj_id`, or an error is raised, and the user gets to specify the type of accumulation:

````sql
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL
);
````

A `.dbmts` file is a superset of SQL, plus derives and configs like `#[...]`.

The above table then will generate the entirety of the remaining boilerplate views, triggers, tables in a `.sql` file. For user interest, they will be able to use `coin_store_hist`, `coin_store_hist_partial`, `coin_store_events`, `coin_store_events_grouped`, `coin_store_events_grouped_partial`.

Note that this is focused on `sqlite3` for now.

# 2 Journal

2025-10-09 Wk 41 Thu - 09:06 +03:00

Spawn [000 Create rust template project](tasks/000%20Create%20rust%20template%20project.md) <a name="spawn-task-49a28a" />^spawn-task-49a28a

2025-10-09 Wk 41 Thu - 11:11 +03:00

Spawn [001 Create basic parsing of derive and es for dbmts](tasks/001%20Create%20basic%20parsing%20of%20derive%20and%20es%20for%20dbmts.md) <a name="spawn-task-2f3be1" />^spawn-task-2f3be1

2025-10-09 Wk 41 Thu - 16:55 +03:00

Spawn [002 Group settings by table and enforce some conditions on the table and settings for dbmts](tasks/002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md) <a name="spawn-task-a09a05" />^spawn-task-a09a05

2025-10-09 Wk 41 Thu - 21:16 +03:00

Spawn [003 Impl writing the expanded event sourcing output given the grouped settings by table for dbmts](tasks/003%20Impl%20writing%20the%20expanded%20event%20sourcing%20output%20given%20the%20grouped%20settings%20by%20table%20for%20dbmts.md) <a name="spawn-task-8f580d" />^spawn-task-8f580d

2025-10-10 Wk 41 Fri - 00:23 +03:00

OK!

2025-10-10 Wk 41 Fri - 08:51 +03:00

Spawn [004 Write dbmts_rs spec](tasks/004%20Write%20dbmts_rs%20spec.md) <a name="spawn-task-b38f20" />^spawn-task-b38f20

# 3 Spawn Trees

* [004 Create dbmts format with event sourcing support](004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)
  * task [000 Create rust template project](tasks/000%20Create%20rust%20template%20project.md)
  * task [001 Create basic parsing of derive and es for dbmts](tasks/001%20Create%20basic%20parsing%20of%20derive%20and%20es%20for%20dbmts.md)
  * task [002 Group settings by table and enforce some conditions on the table and settings for dbmts](tasks/002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md)
    * howto [000 Preserve a last value and use it in iteration mapping](howtos/000%20Preserve%20a%20last%20value%20and%20use%20it%20in%20iteration%20mapping.md)
  * task [003 Impl writing the expanded event sourcing output given the grouped settings by table for dbmts](tasks/003%20Impl%20writing%20the%20expanded%20event%20sourcing%20output%20given%20the%20grouped%20settings%20by%20table%20for%20dbmts.md)
  * paused task [004 Write dbmts_rs spec](tasks/004%20Write%20dbmts_rs%20spec.md)
    * task [005 Write dbmts_rs event sourcing feature spec](tasks/005%20Write%20dbmts_rs%20event%20sourcing%20feature%20spec.md)

# 4 Index

**howto**

[000 Preserve a last value and use it in iteration mapping](howtos/000%20Preserve%20a%20last%20value%20and%20use%20it%20in%20iteration%20mapping.md)

**task**

[000 Create rust template project](tasks/000%20Create%20rust%20template%20project.md)

[001 Create basic parsing of derive and es for dbmts](tasks/001%20Create%20basic%20parsing%20of%20derive%20and%20es%20for%20dbmts.md)

[002 Group settings by table and enforce some conditions on the table and settings for dbmts](tasks/002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md)

[003 Impl writing the expanded event sourcing output given the grouped settings by table for dbmts](tasks/003%20Impl%20writing%20the%20expanded%20event%20sourcing%20output%20given%20the%20grouped%20settings%20by%20table%20for%20dbmts.md)

paused [004 Write dbmts_rs spec](tasks/004%20Write%20dbmts_rs%20spec.md)

[005 Write dbmts_rs event sourcing feature spec](tasks/005%20Write%20dbmts_rs%20event%20sourcing%20feature%20spec.md)
