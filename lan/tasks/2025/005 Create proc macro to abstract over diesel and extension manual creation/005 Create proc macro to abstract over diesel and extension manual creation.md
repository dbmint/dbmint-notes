---
status: todo
---

# 1 Objective

We want to try to keep only one source to create a schema so that it's straightforward to use. We still need to create the underlying database, but then it might be possible to do it from this source instead of a manual creation of a `*.dbmts` file.

We also want to automatically provide common read, write, and delete

# 2 Journal

2025-11-18 Wk 47 Tue - 11:37 +03:00

Spawn [000 Create a proc macro](tasks/000%20Create%20a%20proc%20macro.md) <a name="spawn-task-7ee363" />^spawn-task-7ee363

2025-11-18 Wk 47 Tue - 11:57 +03:00

We should be able to create `*.dbmts` content like this:

````
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL
);
````
