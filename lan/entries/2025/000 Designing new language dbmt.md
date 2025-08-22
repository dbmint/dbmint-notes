---
status: todo
---

#external #docs

# 1 Objective

Notes on `dbmt`, which is a language derivative of `dbml`.

# 2 Journal

## 2.1 Initial Planned Features

2025-07-22 Wk 30 Tue - 18:58

We want the following features in addition to what's in the [dbml syntax docs](https://dbml.dbdiagram.io/docs/) [[#^1]]:

- [ ] Syntax to include other dbmt files

```dbmt
.include "some/path/to/other.dbmt"
```


- [ ] Custom data types

```dbmt
.type HP integer
```

Signals associated with types apply wherever the types are used.

(update)

> [!NOTE] Update
> The content here can be implemented via PartialTable syntax already in dbml. Is it worth adding this to the type syntax if we have syntax for it? Probably not.

Note that even tables may have custom datatypes:

```dmbt
.type SpecialTable Table
```


In case we want to include repetitiveness in the tables, they may be further specified

```dmbt
.type SpecialTable Table {
	id integer [primary key]
	created_on string
}

SpecialTable MySpecialTable {
	myfield integer
}
```

`MySpecialTable` here will automatically have the same fields `id` and `created_on`. 

(/update)

- [ ] Signals with trigger type on table and column

This table is to be put in the specification:

| Trigger  | Event Data     | Meaning                                                                        |
| -------- | -------------- | ------------------------------------------------------------------------------ |
| insert   | table, row     | Occurs whenever a database insertion operation is performed on a table.        |
| update   | table, row     | Occurs whenever a database update operation is performed on a table row.       |
| delete   | table, row     | Occurs whenever a database delete operation is performed on a table row.       |
| write    | table, row     | Occurs on all write operations insert, update, delete.                         |
| read     | table, columns | Occurs when a selection of columns are fetched from a table.                   |
| init     | database       | Occurs on database initialization at the beginning of program execution.       |
| shutdown | database       | Occurs when the database module is issued a shutdown command by user software. |
| cust     | user defined   | User defined trigger. See [Custom Triggers](#331-custom-triggers).             |

| Trigger Type | Meaning                                                                                                                                                                                                                                                                                           |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| notify       | When a trigger in given list occurs, notify the subscriber configured by [User Software](#42-user-software).                                                                                                                                                                                      |
| default      | When a trigger in given list is set to occur, current value associated with event is sent to [User Software](#42-user-software) which must respond with a value to replace it with. This must not chain with `override`.                                                                          |
| override     | When a trigger in given list is set to occur, current value associated with event is sent to [User Software](#42-user-software) which must respond with a value to replace it with. This must not chain with `default`.                                                                           |
| allow        | When a trigger in given list occurs or is set to occur, permission requested to [User Software](#42-user-software), which may respond with `OK (0)` or any other error code that will be issued as an error according to [User Software](#42-user-software) following a `shutdown` trigger event. |
| warn         | When a trigger in given list occurs or is set to occur, filter is requested to [User Software](#42-user-software), which may respond with `Keep (0)` or `Skip (1)` to log a warning according to a mechanism configured by [User Software](#42-user-software).                                    |
| trace        | When a trigger in given list occurs, filter is requested to [User Software](#42-user-software), which may respond with `Keep (0)` or `Skip (1)` to log a trace according to a mechanism configured by [User Software](#42-user-software).                                                         |

(update 0)

2025-08-14 Wk 33 Thu - 13:43

- [ ] Computed Fields

Some fields are computed at some time and shouldn't be manually inserted. A popular example is `created_on` and `updated_on` which are computed on insert and both insert/update events respectively. 

```
Table MyTable {
	%signal get_date_now override=[insert]
	created_on string
	%signal get_date_now override=[update]
	updated_on string
}
```


<details>
<summary> iteration 1</summary>

```dbmt
%signal my_signal_name on=[insert, delete]
```

This applies to the next item, which could be a table or a column.

User should be able to subscribe to these signals and perform some logic on those events.

In addition, signals can be `gated`, meaning that user action is expected before the operation is confirmed. Operation will wait for user to signal OK or ERR.

`gated` signals are specified under `then=` rather than `on=`:

```dbmt
%signal my_signal_name permit=[insert, delete]
```

In addition, we can use `to=` instead of `on=` and `then=`. `to` signals override the event values themselves which are useful for computed fields:

```dmbn
%signal get_current_date to=[write]
created_on string
```

`cust` triggers are custom. The application handles how those events trigger

```
%signal clear_temporary on=[cust(on_cache_clear)]
```

The generated application will subscribe to these signals, when the user publishes to them, they will triggered the configured behavior.

</details>
(/update 0)

- [ ] Virtual Fields

Some fields shouldn't be manually created. They should be computed based on the context on some signal trigger.  We call those virtual fields or virtual columns.

(update)

```dbmt
Table MyTable {
	%virtual
	my_read_only_field integer
	%signal get_current_date to=[write]
	created_on string
}
```

<details>
<summary> iteration 1 </summary>
If the field really shouldn't have a value in the database, and is just a placeholder until it is computed on read, mark it `[computed]`. 

```
Table MyTable {
	%signal(calculate_my_field, via=[read])
	my_read_only_field integer [computed]
	%signal(get_current_date, via=[write])
	created_on string
}
```
</details>
(/update)

- [ ] Data access configuration


| Access Type | Meaning                                                                                                                               |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| fixed       | Table only allows fetch, unless in [admin mode](#47-admin-mode).                                                                      |
| immutable   | Table only allows fetch and insert operations, unless in [admin mode](#47-admin-mode).                                                |
| mutable     | Table only allows fetch, insert, and update operations, unless in [admin mode](#47-admin-mode).                                       |
| ephemeral   | Table allows all operations fetch, insert, update, and delete. This is the only access configuration in [admin mode](#47-admin-mode). |


Those include
- fixed
- immutable
- mutable
- ephemeral

fixed cannot be written at all. immutable only allows inserts. mutable allows inserts and updates but not deletes. and ephemeral allows inserts, updates, and deletes.

Configuration only applies to tables:

(update)

2025-08-14 Wk 33 Thu - 13:02

```dmbn
%immutable
Table MyTable {
	...
}
```

<details>
<summary> iteration 1 </summary>

```dmbn
[access: immutable]
Table MyTable {
	...
}
```

</details>
(/update)

They also apply to table types.

- [ ] Not null by default

All fields cannot be null unless marked explicitly as `[null]`. 

- [ ] Uniqueness checks

Fields marked as `[unique]` will be flagged for that check when a table verification process starts. 

This can be overriden for an expensive check always on read/write with `[always-check]` for that specific field.

(update)

2025-08-14 Wk 33 Thu - 14:25

I'm not sure we want these `[otm]` etc. This is already defined by `dbml` and we should simply use that information.  We need to think about this more, but for now low priority.

Spawn [[#4.1 Designing one-to-many and many-to-many verification checks]] ^spawn-task-44b3b4

<details>
<summary> Iteration 1 </summary>
- [ ] One-to-many or one-to-one

References can be marked `[otm]` for one to many or `[oto]` for one to one. You can also do `[mto]` for many to one. These will be flagged for that check when a table verification process starts.

This can be overriden for an expensive check always on read/write with `[always-check]` for that specific relation.

</details>
(/update)

2025-08-14 Wk 33 Thu - 14:24

- [ ] default symbols


| Symbol       | Effect                                                               |
| ------------ | -------------------------------------------------------------------- |
| get_date_now | Calculates the current date now according to user timezone settings. |

2025-08-21 Wk 34 Thu - 10:44

Table groups already exist as part of dbml. We need


## 2.2 Lower priority things to add later

2025-08-14 Wk 33 Thu - 08:50

For putting things in a logical order, dbml supports [table groups](https://dbml.dbdiagram.io/docs/#tablegroup-notes-1).

As we are a superset of dbmt, we can also give this meaning in Rust database library generation. 

For example, it could be that it forms a namespace.

2025-08-14 Wk 33 Thu - 08:51

dbml supports [partial tables](https://dbml.dbdiagram.io/docs/#tablepartial). This could be a method used for sharing column types across tables. We had plans before about supporting modular table creation in some way. It could also have  semantic relevance for us, because we can recognize all columns under the category of the specified partial table in Rust.

# 3 HowTos

## 3.1 Adding collapsable sections in markdown

- [x] 

From this [gist by pierrejoubert73](https://gist.github.com/pierrejoubert73/902cc94d79424356a8d20be2b382e1ab),

<details>
<summary>Collapsed section! </summary>

Put your content and stuff!
</details>

But this doesn't seem to work well in obsidian. You can open them I guess via `<details open>`  but it still won't show well in obsidian.


But if I do `<!details>` to temporarily break the html syntax I can see it.

# 4 Tasks

## 4.1 Designing one-to-many and many-to-many verification checks

- [ ] 

From [[#^spawn-task-44b3b4]] 

### 4.1.1 Later

# 5 Side Notes

## 5.1 Need to keep heading names unique due to non-unique numbering

2025-08-14 Wk 33 Thu - 09:04

![[Pasted image 20250814090510.png]]

![[Pasted image 20250814090801.png]]

Though it is able to handle non-unique header names by appending a number:

![[Pasted image 20250814090858.png]]

## 5.2 Vscode extensions that that helped me write the specification

2025-08-14 Wk 33 Thu - 17:12

![[Pasted image 20250814171150.png]]![[Pasted image 20250814171150 1.png]]

This helped a lot! I am able to modify markdown files, and reference headings on different files and create auto-sync table of contents really fast with this!

For tables, I still used obsidian and copied the tables over though because they seemed easier to edit this way.

# 6 References
1. [dbml syntax docs][1]

[1]: https://dbml.dbdiagram.io/docs/