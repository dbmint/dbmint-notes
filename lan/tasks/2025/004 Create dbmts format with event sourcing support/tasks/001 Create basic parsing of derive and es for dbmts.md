---
parent: "[[004 Create dbmts format with event sourcing support]]"
spawned_by: "[[004 Create dbmts format with event sourcing support]]"
context_type: task
status: done
---

Parent: [[004 Create dbmts format with event sourcing support]]

Spawned by: [[004 Create dbmts format with event sourcing support]]

Spawned in: [[004 Create dbmts format with event sourcing support#^spawn-task-2f3be1|^spawn-task-2f3be1]]

# 1 Journal

2025-10-09 Wk 41 Thu - 11:14 +03:00

Let's first create a parser that parses the following:

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

So to be minimal with our parsing here, since we only parse settings and then subsequently remove the setting lines in our generated sql,

every `#[...]` setting may chain, so you can have many of them, one per line, until you reach the attached item. Two types of attached items are currently to be supported:

1. Table Creation, Starts with `CREATE TABLE {table_name}`, where case doesn't matter. 
2. Column, Starts with `{column_name} [INTEGER|REAL|TEXT|BLOB]`, where case doesn't matter and the types are as supported by [sqlite.org datatypes](https://sqlite.org/datatype3.html)

2025-10-09 Wk 41 Thu - 12:54 +03:00

Checked `~/src/cloned/gh/LanHikari22/dbmt-py/src/dbmt-py/commands.py` for `Identifier` impl which points to `isidentifier` ([w3schools isidentifier](https://www.w3schools.com/python/ref_string_isidentifier.asp))


2025-10-09 Wk 41 Thu - 14:32 +03:00

We need to ensure that settings attach to immediate targets next, they should not be able to skip their targets on error, and just attach to a different column or table!

2025-10-09 Wk 41 Thu - 15:02 +03:00

(update)
If the user doesn't mark an aggregation for a column, it will not be used.

2025-10-09 Wk 41 Thu - 15:43 +03:00

Everything must be marked, except for `id` and `obj_id`. This should be enforced. It is better to err towards safety than that the user might want to use a hypothetical option in this case. `diffs` is already a table for event sourcing, so there is no need to leave information there that will only get ignored.

(/update)

Every diff table must have `id` and `obj_id`, not null integers, and `id` must be primary key.

2025-10-09 Wk 41 Thu - 15:27 +03:00

Implemented basic parsing. There's some assumptions, we're processing lines, so if the user puts everything under the same line, it won't parse. Technically this should be a char parser, independent of lines.

(Judgment) lines are a convenient assumption, and will be assumed. If needed, this assumption can be relaxed in the future without much cost.

2025-10-09 Wk 41 Thu - 15:45 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
<given>
#[derive(EventSourcing)]
CREATE TABLE 老 (
)
</given>

<expected>
[temp placeholder]
InvalidCreateTableLine(TableNameMustBeAnIdentifier(Empty))
[/temp placeholder]
</expected>

<actual>
InvalidCreateTableLine(MustStartWithCreateTable("#[derive(EventSourcing)]"))
</actual>

thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:475:33:
case-000-000-F-TableNamesMustBeIdentifiers: Output is not as expected
```

```rust
// in fn parse_settings
let table_line = content
	.lines()
	.skip(i)
	.filter(|line| line.starts_with("#["))
	.next()
	.ok_or(FnErr::NoTableCreateLineFound(i, line.to_owned()))?
	.parse::<CreateTableLine>()?;
```

These should skip `i+1`, including the current line.

2025-10-09 Wk 41 Thu - 15:50 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
<given>
#[derive(EventSourcing)]
CREATE TABLE 老 (
)
</given>

<expected>
[temp placeholder]
InvalidCreateTableLine(TableNameMustBeAnIdentifier(Empty))
[/temp placeholder]
</expected>

<actual>
NoTableCreateLineFound(0, "#[derive(EventSourcing)]")
</actual>

thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:475:33:
case-000-000-F-TableNamesMustBeIdentifiers: Output is not as expected
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Also we filter *out*  `#(` lines.

```diff
// in fn parse_settings
-.filter(|line| line.starts_with("#["))
+.filter(|line| !line.starts_with("#["))
```

2025-10-09 Wk 41 Thu - 15:58 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (relevant)
<given>
#[derive(EventSourcing)]
CREATE TABLE 老 (
)
</given>

<expected>
[temp placeholder]
InvalidCreateTableLine(TableNameMustBeAnIdentifier(Empty))
[/temp placeholder]
</expected>

<actual>
InvalidCreateTableLine(TableNameMustBeAnIdentifier(HasSymbols("老")))
</actual>

thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:478:33:
case-000-000-F-TableNamesMustBeIdentifiers: Output is not as expected
```

Yes! That is the error we actually expect.

2025-10-09 Wk 41 Thu - 16:03 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:447:49:
case-001: Parsing should pass: InvalidCreateTableLine(MustStartWithCreateTable("CREATE   TABLE credit_store_diffs "))
```

Right, need to assume that whitespace is variable, unlike in

```rust
// in impl FromStr for CreateTableLine
if !s.to_uppercase().starts_with("CREATE TABLE") {
	return Err(FnErr::MustStartWithCreateTable(s.to_owned()));
}
```

We were already splitting by ascii whitespace, this should be better:

```rust
// in impl FromStr for CreateTableLine
let tokens = s.split_ascii_whitespace().collect_vec();

if tokens[0].to_uppercase() != "CREATE" || tokens[1].to_uppercase() != "TABLE" {
	return Err(FnErr::MustStartWithCreateTable(s.to_owned()));
}
```

2025-10-09 Wk 41 Thu - 16:11 +03:00

One other issue was whitespace, which is captured in the line:

```
<expected>
TableDeriveEventSourcing { table_portion_name: Identifier { s: "credit_store" }, table_line: CreateTableLine { table_name: Identifier { s: "credit_store_diffs" }, line: "CREATE   TABLE credit_store_diffs" } }
</expected

<actual>
TableDeriveEventSourcing { table_portion_name: Identifier { s: "credit_store" }, table_line: CreateTableLine { table_name: Identifier { s: "credit_store_diffs" }, line: "CREATE   TABLE credit_store_diffs " } }
</actual>
```

So we added that to the expected. Now everything is passing so far.

2025-10-09 Wk 41 Thu - 16:34 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
<given>
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL,
)
</given>

<expected>
[TableDeriveEventSourcing { table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" } }, ColumnESSetting { es_aggregate_setting: Latest, table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" }, column_line: ColumnLine { column_name: Identifier { s: "person" }, column_type: Text, nullable: false, pk: false, line: "  person TEXT NOT NULL," } }, ColumnESSetting { es_aggregate_setting: Latest, table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" }, column_line: ColumnLine { column_name: Identifier { s: "coins" }, column_type: Integer, nullable: false, pk: false, line: "  coins INTEGER NOT NULL," } }]
</expected>

<actual>
[TableDeriveEventSourcing { table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" } }]
</actual>

thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:515:29:
case-002: Expected 3 items but got 1
```

2025-10-09 Wk 41 Thu - 16:45 +03:00

The line can be tabbed, so this would fail:

```diff
// in fn parse_settings
-} else if line.starts_with("#[es(") && line.ends_with(")]") {
+} else if line.trim().starts_with("#[es(") && line.ends_with(")]") {
```

2025-10-09 Wk 41 Thu - 16:46 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:508:49:
case-002: Parsing should pass: InvalidESAggregate(Invalid("  latest"))
```

This should also be trimmed:

```diff
// in fn parse_settings
let es_aggregate_setting = line
	.replace("#[es(", "")
	.replace(")]", "")
+   .trim()
	.parse::<ESAggregateSetting>()?;
```

2025-10-09 Wk 41 Thu - 16:49 +03:00

```sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
cargo test

# out (error, relevant)
<given>
#[derive(EventSourcing)]
CREATE TABLE coin_store_diffs (
  id INTEGER NOT NULL PRIMARY KEY,
  obj_id INTEGER NOT NULL,
  #[es(latest)]
  person TEXT NOT NULL,
  #[es(sum)]
  coins INTEGER NOT NULL,
)
</given>

<expected>
ColumnESSetting { es_aggregate_setting: Latest, table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" }, column_line: ColumnLine { column_name: Identifier { s: "coins" }, column_type: Integer, nullable: false, pk: false, line: "  coins INTEGER NOT NULL," } }
</expected>

<actual>
ColumnESSetting { es_aggregate_setting: Sum, table_portion_name: Identifier { s: "coin_store" }, table_line: CreateTableLine { table_name: Identifier { s: "coin_store_diffs" }, line: "CREATE TABLE coin_store_diffs (" }, column_line: ColumnLine { column_name: Identifier { s: "coins" }, column_type: Integer, nullable: false, pk: false, line: "  coins INTEGER NOT NULL," } }
</actual>

thread 'dbmts_parser::tests::parse_settings_tests::test_parse_settings' panicked at src/dbmts_parser.rs:525:33:
case-002: Output is not as expected
```

Oops, the test should expect `Sum` there.

OK! All tests so far pass!

2025-10-09 Wk 41 Thu - 16:54 +03:00

OK