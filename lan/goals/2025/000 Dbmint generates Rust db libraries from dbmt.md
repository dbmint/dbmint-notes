Project description can be found at [000 Dbmint](../../docs/2025/000%20Dbmint.md).

# 1 Goal

Given a `dbmt` schema file, we need to generate a [Rust](https://www.rust-lang.org/) library which integrates that database information into the rust type system and utilize Rust's type system for correctness in library use.

By the end of this goal, we will have a deliverable containing a...

1. [ ] specification document for `dbmt`, a superset language of [`dbml`](https://dbml.dbdiagram.io/docs/). <a name="deliver1" />^deliver1
1. [ ] functional reader and writer for the `dbmt` language <a name="deliver2" />^deliver2
1. [ ] `dbmint` CLI that takes a `dbmt` schema file and creates a Rust library in specified mount location accordingly. <a name="deliver3" />^deliver3
1. [ ] Rust library generated via `dbmt` schema file allowing...
   1. [ ] type-correct selections, queries, inserts, updates, deletions, and basic star joins <a name="deliver4-1" />^deliver4-1
   1. [ ] db verification including at the granular level (per column, per table) and the generic level (per type). Those should be routed in the `dbmt` schema file, and implemented either in the library for built-in commands or by the library user via library signals. They can distinguish different events such as writing, reading, deleting, inserting, init, etc. <a name="deliver4-2" />^deliver4-2
   1. [ ] Data access hygiene with tables marked Fixed, Immutable, Mutable, or Ephemeral. These correspond to operations (Read, Read/Insert, Read/Insert/Update, Read/Insert/Update/Delete). This option may be bypassed when generating an admin use library. <a name="deliver4-3" />^deliver4-3
   1. [ ] auto-computed fields on events like creation (example: date of creation), and writing (example: last update date). <a name="deliver4-4" />^deliver4-4
   1. [ ] virtual read-only fields that do not exist really in the database but act as if they do and are computed on fetch. <a name="deliver4-5" />^deliver4-5
1. [ ] Extended visualization tools from `dbml` for `dbmt` including standalone nodejs rendering and vscode extension. <a name="deliver5" />^deliver5
1. [ ] Basic database editing workflow solution that allow (1) a logical order of inserting new table entries, and (2) a custom table for insertion and editing purposes that corresponds to the schema details. <a name="deliver6" />^deliver6
1. [ ] Tests proving that the generated rust library...
   1. [ ] Works at scale with respect to number of columns in tables <a name="deliver7-1" />^deliver7-1
   1. [ ] Works at scale with respect to the number of tables in a database <a name="deliver7-2" />^deliver7-2
   1. [ ] Works at scale with respect to the number of defined table joins <a name="deliver7-3" />^deliver7-3
   1. [ ] Provides efficient database operations proven by measurements <a name="deliver7-4" />^deliver7-4
1. [ ] Mechanism of converting a `dbmt` file to a structurally equivalent `dbml` file, at least for the purposes of generating a sqlite3 database <a name="deliver8" />^deliver8

# 2 Objectives
