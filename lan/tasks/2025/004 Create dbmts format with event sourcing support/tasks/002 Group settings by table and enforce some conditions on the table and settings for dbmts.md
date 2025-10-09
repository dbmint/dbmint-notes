---
parent: "[[004 Create dbmts format with event sourcing support]]"
spawned_by: "[[004 Create dbmts format with event sourcing support]]"
context_type: task
status: done
---

Parent: [[004 Create dbmts format with event sourcing support]]

Spawned by: [[004 Create dbmts format with event sourcing support]]

Spawned in: [[004 Create dbmts format with event sourcing support#^spawn-task-a09a05|^spawn-task-a09a05]]

# 1 Objective

Now that the basic parsing is done, we need to ensure that 

1. No duplicate `#[es(latest|sum)]`  settings exist for the same column
2. No duplicate `#[derive(EventSourcing)]` settings exist for the same table
3. Natural order, in which `#[derive(EventSourcing)]` is always followed by `#[es(latest|sum)]` settings for the same table.
4. All columns expect for `id` and `obj_id` have the `#[es(latest|sum)]` setting for all tables with the `#[derive(EventSourcing)]` setting.
5. All tables with the `#[derive(EventSourcing)]` setting must have a non-null primary key integer `id` and non-null integer `obj_id`. 
6. `id` and `obj_id` must have no `#[es(latest|sum)]` settings.

# 2 Journal

2025-10-09 Wk 41 Thu - 17:10 +03:00

There's still some variation on the columns that we haven't captured yet:

Example:

```sql
opt_diff_id INTEGER NULL REFERENCES coin_store_diffs(id),
ev_action TEXT CHECK(ev_action IN ('insert', 'update', 'delete', 'open', 'close', 'reopen')) NOT NULL,
```

from

```sql
CREATE TABLE coin_store_events (
  id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
  opt_diff_id INTEGER NULL REFERENCES coin_store_diffs(id),
  ev_action TEXT CHECK(ev_action IN ('insert', 'update', 'delete', 'open', 'close', 'reopen')) NOT NULL,
  span INTEGER NOT NULL,
  frame INTEGER NOT NULL,
  created_on_ts REAL NOT NULL,
  ev_desc TEXT NOT NULL
);
```

The `REFERENCES` and the `CHECK`.  We likely won't need them for for event sourcing derives though.

2025-10-09 Wk 41 Thu - 17:35 +03:00

```rust
impl From<&[Setting]> for SettingsByTable {
    fn from(value: &[Setting]) -> Self {
        todo!()
    }
}
```

Why doesn't this have an `Err` type...

We'll just create our own trait for this

Oh! I thought to name it `TryFrom` but it exists:

```rust
impl TryFrom<&[Setting]> for SettingsByTable {
    type Error = SettingsByTableTryFromError;

    fn try_from(value: &[Setting]) -> Result<Self, Self::Error> {
        todo!()
    }
}
```

2025-10-09 Wk 41 Thu - 18:03 +03:00

![[Pasted image 20251009180337.png]]

Hmm, I guess I can't implement `TryFrom` to give many in this way.

![[Pasted image 20251009180446.png]]

Slices are always foreign!

```rust
struct SettingsByTable {
    groups: Vec<(TableDeriveEventSourcingSetting, Vec<ColumnESSetting>)>,
}
```

This can work though with just `TryFrom`. 

2025-10-09 Wk 41 Thu - 18:07 +03:00

These error variants should match our objective constraints:

```rust
#[derive(Error, Debug)]
pub enum SettingsByTableTryFromError {
    #[error("Columns must have distinct settings: {0:?}")]
    DuplicateESSettingForColumn(Setting),

    #[error("Tables must have distinct derives: {0:?}")]
    DuplicateESDeriveSettingForTable(Setting),

    #[error("A Table ES Derive must only have Column ES Settings of its table followed: {0:?}")]
    BrokenOrderForESDeriveThenSettings(Setting),

    #[error("id and obj_id columns must exist in the given table with EventSourcing derive")]
    IdAndObjIdMustBePresent,

    #[error("id and obj_id cannot be aggregated")]
    IdAndObjIdMustHaveNoSettings,

    #[error("All table columns must have an ES setting besides id and obj_id")]
    AllTableColumnsExceptIdAndObnIdMustHaveESSettings,
}
```

When `SettingsByTable` is constructed, we then know (on correct impl) that all these constraints are met.

2025-10-09 Wk 41 Thu - 18:19 +03:00

We can group by a condition using [docs.rs itertools chunk_by](https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.chunk_by). But we want to group by table_name + column_name and count that each group has 1 element to check `DuplicateESSettingForColumn`.  

2025-10-09 Wk 41 Thu - 18:29 +03:00

It seems maybe we want [py more-itertools bucket](https://more-itertools.readthedocs.io/en/v10.2.0/api.html#more_itertools.bucket) ([docs.rs more-itertools bucket](https://docs.rs/more-itertools/0.1.6/more_itertools/grouping/bucket/fn.bucket.html))

Though the type of `K` is general for [docs.rs itertools chunk_by](https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.chunk_by). So the boolean grouping is likely just one use case. I could return the tuple of interest to group by, and all elements that match the same result `K` then should be in the same group.

2025-10-09 Wk 41 Thu - 18:52 +03:00

Like this!

```rust
// in impl TryFrom<&[Setting]> for SettingsByTable
let columns_have_distinct_settings = settings
	.iter()
	.chunk_by(|k| match k {
		Setting::ColumnESSetting(column_essetting) => {
			Some((
				&column_essetting.table_line.table_name,
				&column_essetting.column_line.column_name,
			))
		},
		_ => None,
	})
	.into_iter()
	.all(|(key, group)| {
		match key {
			Some(..) => {
				group.count() == 1
			},
			None => true,
		}
	});

if !columns_have_distinct_settings {
	return Err(FnErr::ColumnSettingsMustBeDistinct)
}
```

2025-10-09 Wk 41 Thu - 18:56 +03:00

We don't have to clone in this case

```diff
// in impl TryFrom<&[Setting]> for SettingsByTable
-column_essetting.table_line.table_name.clone(),
+&column_essetting.table_line.table_name,
-column_essetting.column_line.column_name.clone(),
+&column_essetting.column_line.column_name,
```

2025-10-09 Wk 41 Thu - 19:10 +03:00

We could check consecutive indices with windows for the order constraint

2025-10-09 Wk 41 Thu - 19:13 +03:00

Like this!

```rust
// in impl TryFrom<&[Setting]> for SettingsByTable
let settings_of_the_same_table_are_consecutive = settings
	.iter()
	.enumerate()
	.chunk_by(|(_, k)| {
		match k {
			Setting::TableDeriveEventSourcing(setting) => &setting.table_line.table_name,
			Setting::ColumnESSetting(setting) => &setting.table_line.table_name,
		}
	})
	.into_iter()
	.all(|(_, group)| {
		group
			.into_iter()
			.tuple_windows::<(_, _)>()
			.all(|((i0, _), (i1, _))| i1 - i0 == 1)
	});
	
if !settings_of_the_same_table_are_consecutive {
	return Err(FnErr::SettingsOfSameTableAreConsecutive)
}
```

2025-10-09 Wk 41 Thu - 19:18 +03:00

```rust
// in pub enum SettingsByTableTryFromError
#[error("id and obj_id columns must exist in the given table with EventSourcing derive")]
IdAndObjIdMustBePresent,

#[error("id and obj_id cannot be aggregated")]
IdAndObjIdMustHaveNoSettings,

#[error("All table columns must have an ES setting besides id and obj_id")]
AllTableColumnsExceptIdAndObnIdMustHaveESSettings,
```

We can't check these constraints when grouping... We don't have the whole content.  But we can change that:

```diff
-impl TryFrom<&[Setting]> for SettingsByTable {
impl TryFrom<(&[Setting], &str)> for SettingsByTable {
```

2025-10-09 Wk 41 Thu - 19:42 +03:00

Spawn [[000 Preserve a last value and use it in iteration mapping]] ^spawn-howto-ecf5e1

2025-10-09 Wk 41 Thu - 20:30 +03:00

![[Pasted image 20251009203033.png]]

![[Pasted image 20251009203040.png]]

Weird issue where the presence of the filter makes intellisense think the type now is unknown. Maybe because the pipeline is pretty long?

It works fine after collecting

![[Pasted image 20251009204248.png]]

2025-10-09 Wk 41 Thu - 21:02 +03:00

Okay! All conditions have been checked!

```rust
pub struct SettingsByTable {
    pub chunked: Vec<(Identifier, Vec<Setting>)>,
}
```

This is more natural for the groups since `chunked_by` by table name can give us this.

2025-10-09 Wk 41 Thu - 21:12 +03:00

```rust
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let chunked = settings
	.iter()
	.chunk_by(|k| {
		match k {
			Setting::TableDeriveEventSourcing(setting) => setting.table_line.table_name.clone(),
			Setting::ColumnESSetting(setting) => setting.table_line.table_name.clone(),
		}
	})
	.into_iter()
	.map(|(table_name, grp)| {
		(
			table_name, 
			grp
				.into_iter()
				.map(|setting| setting.clone())
				.collect_vec()
		)
	})
	.collect::<Vec<_>>();

Ok(Self {
	chunked,
})
```

2025-10-09 Wk 41 Thu - 21:14 +03:00

OK!

2025-10-09 Wk 41 Thu - 22:46 +03:00

Came from [[003 Impl writing the expanded event sourcing output given the grouped settings by table for dbmts]] for testing!

```sh
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
```

2025-10-09 Wk 41 Thu - 22:51 +03:00

```diff
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let all_selected_tables_have_id_and_obj_id = selected_tables_and_columns_grouped_by_table
	.iter()
	.all(|(_, grp)| {
		grp
			.iter()
			.any(|(_, column_line)| {
+				log::trace!("column_name: {:?}", column_line.column_name);
				column_line.column_name.s == "id" ||
				column_line.column_name.s == "obj_id"
			})
	});
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[2025-10-09T19:49:08Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "id" }
[2025-10-09T19:49:08Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "person" }

thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: IdAndObjIdMustBePresent
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

2025-10-09 Wk 41 Thu - 22:56 +03:00

```diff
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let tables_and_columns_grouped_by_table = content
	[...]
	.map(|(opt_table_name, grp)| {
		match opt_table_name {
			None => None,
			Some(table_name) => {
				Some((
					table_name, 
					grp
						.into_iter()
						.map(|(opt_table_line, opt_column_line)| {
							if let Some(table_line) = opt_table_line && let Some(column_line) = opt_column_line {
+								log::trace!("column_name1: {:?}", column_line.column_name);
								Some((table_line, column_line))
							} else {
								None
							}
						})
						.flatten()
						.collect::<Vec<_>>()
				))
			},
		}
	})
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "id" }
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "obj_id" }
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "person" }
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "coins" }
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "id" }
[2025-10-09T19:55:51Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "person" }

thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: IdAndObjIdMustBePresent
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

2025-10-09 Wk 41 Thu - 23:05 +03:00

Did our attempt to propagate the table line to all rows fail?

```diff
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let tables_and_columns_grouped_by_table = content
	[...]
	.scan(None, |state, (opt_table_line, opt_column_line)| {
		if let Some(table_line) = opt_table_line {
			*state = Some(table_line);
		}
	
+		log::trace!("scanned table_line: {:?}", state);
+		log::trace!("scanned column_line: {:?}", opt_column_line);
		Some((state.clone(), opt_column_line))
	})
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "id" }, column_type: Integer, nullable: false, pk: true, autoincrement: false, line: "  id INTEGER NOT NULL PRIMARY KEY," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "obj_id" }, column_type: Integer, nullable: false, pk: false, autoincrement: false, line: "  obj_id INTEGER NOT NULL," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "id" }
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "obj_id" }
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "person" }, column_type: Text, nullable: false, pk: false, autoincrement: false, line: "  person TEXT NOT NULL," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "person" }
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "coins" }, column_type: Integer, nullable: false, pk: false, autoincrement: false, line: "  coins INTEGER NOT NULL" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned table_line: Some(CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: None
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "coins" }
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "id" }
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "person" }

thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: IdAndObjIdMustBePresent
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

They are preserved...

```sh
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "id" }, column_type: Integer, nullable: false, pk: true, autoincrement: false, line: "  id INTEGER NOT NULL PRIMARY KEY," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "obj_id" }, column_type: Integer, nullable: false, pk: false, autoincrement: false, line: "  obj_id INTEGER NOT NULL," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "person" }, column_type: Text, nullable: false, pk: false, autoincrement: false, line: "  person TEXT NOT NULL," })
[2025-10-09T20:06:20Z TRACE dbmts_rs::dbmts_parser] scanned column_line: Some(ColumnLine { column_name: Identifier { s: "coins" }, column_type: Integer, nullable: false, pk: false, autoincrement: false, line: "  coins INTEGER NOT NULL" })
```

So the condition that both are `Some` should yield us all of the columns.

2025-10-09 Wk 41 Thu - 23:17 +03:00

Well we should expect that since `column_name1` had no issue passing on, and they're `Some` on both.

2025-10-09 Wk 41 Thu - 23:23 +03:00

This might just be because `all` shortcircuited at the first failure.

2025-10-09 Wk 41 Thu - 23:25 +03:00

```rust
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let all_selected_tables_have_id_and_obj_id = selected_tables_and_columns_grouped_by_table
	.iter()
	.all(|(table_name, grp)| {
		log::trace!("table_name: {table_name:?}");
		grp
			.iter()
			.filter(|(_, column_line)| {
				log::trace!("column_name: {:?}", column_line.column_name);
				column_line.column_name.s == "id" ||
				column_line.column_name.s == "obj_id"
			})
			.count()
			.pipe(|count| {
				count == 2
			})
	});
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "id" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "obj_id" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "person" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "coins" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] table_name: Identifier { s: "coin_store_diffs" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "id" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "obj_id" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] table_name: Identifier { s: "coin_store_diffs" }
[2025-10-09T20:26:21Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "person" }

thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: IdAndObjIdMustBePresent
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

There should only be one table, and one group!

2025-10-09 Wk 41 Thu - 23:29 +03:00

```diff
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let tables_and_columns_grouped_by_table = content
	[...]
	.map(|(opt_table_name, grp)| {
+    	log::trace!("opt_table_name1: {opt_table_name:?}");
		match opt_table_name {
			None => None,
			Some(table_name) => {
				Some((
					table_name, 
					grp
						.into_iter()
						.map(|(opt_table_line, opt_column_line)| {
							if let Some(table_line) = opt_table_line && let Some(column_line) = opt_column_line {
								log::trace!("column_name1: {:?}", column_line.column_name);
								Some((table_line, column_line))
							} else {
								None
							}
						})
						.flatten()
						.collect::<Vec<_>>()
				))
			},
		}
})
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: None
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: Some(Identifier { s: "coin_store_diffs" })
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "id" }
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "obj_id" }
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: None
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: Some(Identifier { s: "coin_store_diffs" })
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "person" }
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: None
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: Some(Identifier { s: "coin_store_diffs" })
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "coins" }
[2025-10-09T20:31:19Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: None
```

It's creating 3 groups!

Really we expect that to trigger at most 2 times, not 7.

2025-10-09 Wk 41 Thu - 23:44 +03:00

We filtered all instances of `(None, None)` in the beginning, and the issue resolves:

```rust
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let tables_and_columns_grouped_by_table = content
	.lines()
	.map(|line| {
		if let Ok(create_table_line) = line.parse::<CreateTableLine>() {
			(Some(create_table_line), None)
		} else if let Ok(column_line) = line.parse::<ColumnLine>() {
			(None, Some(column_line))
		} else {
			(None, None)
		}
	})
	.filter(|(opt_table_line, opt_column_line)| {
		opt_table_line.is_some() || opt_column_line.is_some()
	})
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: None
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] opt_table_name1: Some(Identifier { s: "coin_store_diffs" })
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "id" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "obj_id" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "person" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name1: Identifier { s: "coins" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] table_name: Identifier { s: "coin_store_diffs" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "id" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "obj_id" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "person" }
[2025-10-09T20:43:53Z TRACE dbmts_rs::dbmts_parser] column_name: Identifier { s: "coins" }

thread 'main' panicked at src/main.rs:38:14:
Failed to group settings by table: AllTableColumnsExceptIdAndObjIdMustHaveESSettings
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

2025-10-09 Wk 41 Thu - 23:48 +03:00

We have to filter out `obj_id` and `id`

```diff
// in impl TryFrom<(&[Setting], &str)> for SettingsByTable
let all_other_columns_have_settings = selected_tables_and_columns_grouped_by_table
	.iter()
	.all(|(_, grp)| {
		grp
			.iter()
+			.filter(|(_, column_line)| {
+				column_line.column_name.s != "id" &&
+				column_line.column_name.s != "obj_id"
+			})
			.all(|(table_line, column_line)| {
				settings
					.iter()
					.find(|setting| {
						match setting {
							Setting::ColumnESSetting(setting) => {
								setting.table_line.table_name == table_line.table_name &&
								setting.column_line.column_name == column_line.column_name
							},
							_ => false,
						}
					})
					.is_some()
			})
	});
```

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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

# out (error)
[no error]
```

2025-10-09 Wk 41 Thu - 23:49 +03:00

It generates!

There's `{LATEST_COLUMNS}` that didn't get replaced because I changed it to `{LATEST_COLUMNS_LIST}` but missed some.

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL
);

INSERT INTO coin_store_diffs (id, obj_id, person, coins) VALUES 
  (1, 1, 'person001', 10),
  (2, 1, 'Z', 15),
  (3, 1, 'A', 5),
  (4, 2, 'person002', 7),
  (5, 2, 'person002', 14),
  (6, 3, 'person003', 8),
  (7, 1, 'C', 0)
;

INSERT INTO coin_store_events (opt_diff_id, ev_action, span, frame, created_on_ts, ev_desc) VALUES
  (null, 'open', 1, 1, 1758935304.168087602, 'New Years Eve'),
  (1, 'insert', 1, 1, 1758935465.865269566, ''),
  (2, 'update', 1, 1, 1758935520.052601247, ''),
  (3, 'update', 1, 1, 1758935543.867800435, ''),
  (null, 'open', 2, 1, unixepoch('subsec') + 0.1, 'New Years Eve Plan 1'),
  (4, 'insert', 2, 1, unixepoch('subsec') + 0.2, ''),
  (5, 'update', 2, 1, unixepoch('subsec') + 0.3, ''),
  (null, 'close', 2, 1, unixepoch('subsec') + 0.4, 'New Years Eve Plan 1'),
  (6, 'insert', 1, 1, unixepoch('subsec') + 0.5, ''),
  (7, 'delete', 1, 1, unixepoch('subsec') + 0.6, ''),
  (null, 'open', 2, 2, unixepoch('subsec') + 0.7, 'New Years Eve Plan 2'),
  (null, 'open', 1, 2, unixepoch('subsec') + 0.8, 'Birthday Party')
;

DELETE FROM coin_store_events_grouped_partial;

INSERT INTO coin_store_events_grouped_partial
SELECT * 
FROM coin_store_events_grouped
WHERE obj_id <> 1;

.save "events.db"
EOF
) > events.sql
```

2025-10-09 Wk 41 Thu - 23:54 +03:00

Ouch

```
thread 'main' panicked at src/dbmts_parser.rs:77:18:
index out of bounds: the len is 0 but the index is 0
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

I put the token indexing before the length check...

2025-10-09 Wk 41 Thu - 23:57 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cat events.sql | sqlite3

# out (error)
Parse error near line 2: near "CREATE": syntax error
  d_on_ts REAL NOT NULL,   ev_desc TEXT NOT NULL )  CREATE TABLE coin_store_even
                                      error here ---^
Parse error near line 30: near "ev_desc": syntax error
   person TEXT NOT NULL,   coins INTEGER NOT NULL   ev_desc TEXT NOT NULL );
                                      error here ---^
Parse error near line 48: near "CREATE": syntax error
    person TEXT NOT NULL,   coins INTEGER NOT NULL  CREATE TABLE coin_store_hist
                                      error here ---^
Parse error near line 165: no such table: main.coin_store_events
Parse error near line 176: no such table: main.coin_store_events
Parse error near line 187: no such table: main.coin_store_events_grouped_partial
```

2025-10-09 Wk 41 Thu - 23:59 +03:00

```diff
pub fn strip_settings(content: &str) -> String {
    content
        .lines()
-       .filter(|line| line.starts_with("#["))
+       .filter(|line| !line.trim().starts_with("#["))
        .join("\n")
}
```

Ok I need to have the `INSERT INTO` be at the end, but with the preproccessing here they wouldn't be. Let's just remove them and add them on our own.

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo run -- -vv to_sql <(cat << 'EOF'
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
) > events.sql

echo -e "$(cat << 'EOF'
INSERT INTO coin_store_diffs (id, obj_id, person, coins) VALUES 
  (1, 1, 'person001', 10),
  (2, 1, 'Z', 15),
  (3, 1, 'A', 5),
  (4, 2, 'person002', 7),
  (5, 2, 'person002', 14),
  (6, 3, 'person003', 8),
  (7, 1, 'C', 0)
;

INSERT INTO coin_store_events (opt_diff_id, ev_action, span, frame, created_on_ts, ev_desc) VALUES
  (null, 'open', 1, 1, 1758935304.168087602, 'New Years Eve'),
  (1, 'insert', 1, 1, 1758935465.865269566, ''),
  (2, 'update', 1, 1, 1758935520.052601247, ''),
  (3, 'update', 1, 1, 1758935543.867800435, ''),
  (null, 'open', 2, 1, unixepoch('subsec') + 0.1, 'New Years Eve Plan 1'),
  (4, 'insert', 2, 1, unixepoch('subsec') + 0.2, ''),
  (5, 'update', 2, 1, unixepoch('subsec') + 0.3, ''),
  (null, 'close', 2, 1, unixepoch('subsec') + 0.4, 'New Years Eve Plan 1'),
  (6, 'insert', 1, 1, unixepoch('subsec') + 0.5, ''),
  (7, 'delete', 1, 1, unixepoch('subsec') + 0.6, ''),
  (null, 'open', 2, 2, unixepoch('subsec') + 0.7, 'New Years Eve Plan 2'),
  (null, 'open', 1, 2, unixepoch('subsec') + 0.8, 'Birthday Party')
;

DELETE FROM coin_store_events_grouped_partial;

INSERT INTO coin_store_events_grouped_partial
SELECT * 
FROM coin_store_events_grouped
WHERE obj_id <> 1;

.save "events.db"

EOF
)" >> events.sql
```

2025-10-10 Wk 41 Fri - 00:05 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cat events.sql | sqlite3

# out
Parse error near line 7: near "CREATE": syntax error
  d_on_ts REAL NOT NULL,   ev_desc TEXT NOT NULL )  CREATE TABLE coin_store_even
                                      error here ---^
Parse error near line 35: near "ev_desc": syntax error
   person TEXT NOT NULL,   coins INTEGER NOT NULL   ev_desc TEXT NOT NULL );
                                      error here ---^
Parse error near line 53: near "CREATE": syntax error
    person TEXT NOT NULL,   coins INTEGER NOT NULL  CREATE TABLE coin_store_hist
                                      error here ---^
Parse error near line 170: no such table: main.coin_store_events
Parse error near line 181: no such table: main.coin_store_events
Parse error near line 192: no such table: main.coin_store_events_grouped_partial
Parse error near line 212: no such table: coin_store_events
Parse error near line 227: no such table: coin_store_events_grouped_partial
Parse error near line 229: no such table: coin_store_events_grouped_partial
```

We didn't put a semicolon to the creation of the `{TABLE_NAME}_events` table

2025-10-10 Wk 41 Fri - 00:19 +03:00

It works! It reproduces the experiment! For just this one table, the derives write 196 extra SQL lines behind the scenes!