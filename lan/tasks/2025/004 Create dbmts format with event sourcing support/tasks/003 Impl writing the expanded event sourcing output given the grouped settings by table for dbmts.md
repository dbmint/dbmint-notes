---
parent: '[[004 Create dbmts format with event sourcing support]]'
spawned_by: '[[004 Create dbmts format with event sourcing support]]'
context_type: task
status: done
---

Parent: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned by: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned in: [<a name="spawn-task-8f580d" />^spawn-task-8f580d](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md#spawn-task-8f580d)

# 1 Journal

2025-10-09 Wk 41 Thu - 21:18 +03:00

2025-10-09 Wk 41 Thu - 22:22 +03:00

Implemented `sql_derive_event_sourcing`! Everything falls into place once we do parsing and grouping well!

Now to test it, a lot of things are happening.

(HowTo) Checked `~/src/cloned/gh/deltachives/2025-Wk37-000-obsidian-migration/src/bin/app.rs` for argparsing. Also got `read_file_content`.

2025-10-09 Wk 41 Thu - 22:36 +03:00

Implemented the driver!

(HowTo) Checked `001 pulldown cmark to cmark escapes first obsidian tag on writeback` on `lan-setup-notes` for multiline string to file inline in the command

````sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run to_sql <(cat << 'EOF'
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  person TEXT NOT NULL,
  coins INTEGER NOT NULL
);
EOF
)

# out
[nothing]
````

It seems the string is empty, so it prints nothing.

Right, we meant this:

````sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run to_sql <(cat << 'EOF'
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL
);
EOF
)

# out (error, relevant)
thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: IdAndObjIdMustBePresent
````

It is present. Let's continue testing these in [002 Group settings by table and enforce some conditions on the table and settings for dbmts](002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md)

2025-10-10 Wk 41 Fri - 00:22 +03:00

OK!
