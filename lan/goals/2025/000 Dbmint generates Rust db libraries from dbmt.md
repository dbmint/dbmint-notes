
Project description can be found at [[000 Dbmint]].


# 1 Goal

Given a `dbmt` schema file, we need to generate a [Rust](https://www.rust-lang.org/) library which integrates that database information into the rust type system and utilize Rust's type system for correctness in library use.

By the end of this goal, we will have a deliverable containing a...

1.  [ ] specification document for `dbmt`, a superset language of [`dbml`](https://dbml.dbdiagram.io/docs/). ^deliver1
2.  [ ] functional reader and writer for the `dbmt` language ^deliver2
3.  [ ] `dbmint` CLI that takes a `dbmt` schema file and creates a Rust library in specified mount location accordingly. ^deliver3
4.  [ ] Rust library generated via `dbmt` schema file allowing...
	1.  [ ] type-correct selections, queries, inserts, updates, deletions, and basic star joins ^deliver4-1
	2.  [ ] db verification including at the granular level (per column, per table) and the generic level (per type). Those should be routed in the `dbmt` schema file, and implemented either in the library for built-in commands or by the library user via library signals. They can distinguish different events such as writing, reading, deleting, inserting, init, etc. ^deliver4-2
	3. [ ] Data access hygiene with tables marked Fixed, Immutable, Mutable, or Ephemeral. These correspond to operations (Read, Read/Insert, Read/Insert/Update, Read/Insert/Update/Delete). This option may be bypassed when generating an admin use library. ^deliver4-3
	4. [ ] auto-computed fields on events like creation (example: date of creation), and writing (example: last update date). ^deliver4-4
	5. [ ] virtual read-only fields that do not exist really in the database but act as if they do and are computed on fetch. ^deliver4-5
5. [ ] Extended visualization tools from `dbml` for `dbmt` including standalone nodejs rendering and vscode extension. ^deliver5
6. [ ] Basic database editing workflow solution that allow (1) a logical order of inserting new table entries, and (2) a custom table for insertion and editing purposes that corresponds to the schema details. ^deliver6
7. [ ] Tests proving that the generated rust library...
	1. [ ] Works at scale with respect to number of columns in tables ^deliver7-1
	2. [ ] Works at scale with respect to the number of tables in a database ^deliver7-2
	3. [ ] Works at scale with respect to the number of defined table joins ^deliver7-3
	4. [ ] Provides efficient database operations proven by measurements ^deliver7-4
8. [ ] Mechanism of converting a `dbmt` file to a structurally equivalent `dbml` file, at least for the purposes of generating a sqlite3 database ^deliver8

# 2 Objectives