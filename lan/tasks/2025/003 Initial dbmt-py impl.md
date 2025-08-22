
### 0.1.1 Pend

# 1 Objective

2025-08-14 Wk 33 Thu - 19:35

Here's the plan to a functioning `dbmt-py` implementation and release on PyPi:

This is anchored by the [dbmt spec](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md).

- [ ] Partially parse a `dbmt` script, removing dot and percent commands, and validate the underlying `dbml` using `PyDBML`.
	- [ ] Test with just basic dot and percent syntax parsing that we have a valid `dbml`.
	- [ ] Test with complete `dbmt` parsing that the `dbmt` file is valid AND the `dbml` file is valid.
	- [ ] Test going full circle `dbmt` -> `dbml` -> `dbmt`.
- [ ] Implement/test parsing for `%virtual`, `%ensure_unique`, `%ensure_join_relation`, `%fixed`, `%immutable`, `%mutable`, `%ephemeral` command parsing
	- [ ] Test `%virtual`, `%ensure_unique`, `%ensure_join_relation` can only be applied to columns.
	- [ ] Test `%ensure_unique` can only work for a column with `[unique]` configuration.
	- [ ] Test `%fixed`, `%immutable`, `%mutable`, `%ephemeral` can only apply to tables.
	- [ ] Test only one of `%fixed`, `%immutable`, `%mutable`, `%ephemeral`  may apply to a table.
- [ ] implement/test parsing of `.type` command, which can have configuration of `%signal`. 
- [ ] Implement/test `%signal` command parsing
	- [ ] Test `%signal` may only apply to `.type`, tables, and columns.
	- [ ] Test that symbols defined cannot repeat
	- [ ] Test that each item can have only one symbol defined per specified trigger on trigger types other than `notify`.
	- [ ] Test that multiple symbols may be defined on trigger type `notify`.
	- [ ] Test that only the following trigger types are allowed: `notify`, `allow`, `warn`, `trace`, `override`, `default`.
	- [ ] Test that only the following triggers are allowed without using `cust(...)`: `insert`, `update`, `delete`, `write`, `read`, `init`, `shutdown`,
	- [ ] Test valid use of `cust({symbol})` by ensuring that `{symbol}` is a C-like token.
- [ ] Implement `.include` command by creating an include tree of `dbmt` files.
	- [ ] Test being able to combine multiple files into a single `dbmt` file.
	- [ ] Test that single `dmbt`  file is a valid `dmbt` file.
- [ ] Generate a `JSON` representation of a given `dmbt` file for interoperability with the Rust library generation code.
	- [ ] Test going full circle `dmbt` -> `JSON` -> `dbmt`.

^impl-dbmt-py-objectives
# 2 Journal

From [[001 Create Python and Rust parsing for dbmt language#^spawn-task-060600|^spawn-task-060600]] in [[001 Create Python and Rust parsing for dbmt language]]

2025-08-06 Wk 32 Wed - 06:01

2025-08-14 Wk 33 Thu - 15:52

Let's create the project!

2025-08-14 Wk 33 Thu - 16:23

We need a python3 library project template. Something similar to [checkpipe](https://github.com/LanHikari22/checkpipe). with a `requirements.txt` and `pyproject.toml`. 

Spawn [[#3.1 Create a python3 library project template]] ^spawn-task-a2863a

2025-08-14 Wk 33 Thu - 19:02

Awesome, we have a template python3 library. Let's create the project!

```sh
cp -r ~/src/cloned/gh/LanHikari22/lan-exp-scripts/templates/2025/topics/py3/persistant/001-lan-library/library ~/src/cloned/gh/LanHikari22/dbmt-py
```

Now we need to deal with all the TODOs.

```sh
# in /home/lan/src/cloned/gh/LanHikari22/dbmt-py
grep 'TODO' *       

# out
mypy.sh:app_name="TODO app name"
pyproject.toml:name = "TODO Project Name"
pyproject.toml:description = "TODO Description"
README.md:  TODO library
README.md:  <p>TODO description</p>
README.md:- [Why TODO My tool?](#why-todo-my-tool)
README.md:## Why TODO My tool?
README.md:pip install TODO my tool
README.md:TODO How do you use the tool?
README.md:TODO Thank people!
```

Some of these are included in a heading, so need to make sure to regenerate the Table of Contents.

There's also the library name itself, `todo_library_name`. 

Make sure to include `pydbml` as a dependency in both `pyproject.toml` and `requirements.txt`. 


2025-08-15 Wk 33 Fri - 08:07

I ran into an interesting use case:

```sh
%signal get_date_now override=[insert]
%signal is_valid_timestamp allow=[insert, update]
.type timestamp_t varchar
```

Right now this is not allowed, because you have two non-notify subscribers on `insert`. But this use case makes sense, it needs a priority concept:  `override` -> `allow` -> {`warn`,  `trace`}

With the priority concept, it is possible for multiple subscribers to exist, just not on the same trigger type, and their execution will follow this order. Then we could have fields that get computed, then get validated, and if valid, they may warn or trace.

Note that `notify` isn't here because that's top priority on its own. 

 Internally, this also could be chained synchronously on the database side. It makes computes `get_date_now` (provided symbol), but then makes requests to `is_valid_timestamp` afterwards to user software.

2025-08-15 Wk 33 Fri - 08:37

Done, see [Signal Trigger Type priority queue](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#343-signal-trigger-type-priority-queue) in the spec.

2025-08-15 Wk 33 Fri - 09:03

Added also a `default` as a signal trigger type which is exclusive with `override`, to give the user the option to bypass a computed field. This would be a field that would be computed only if the user decides not to provide a value.

2025-08-15 Wk 33 Fri - 10:40

Spawn [[001 Create Python and Rust parsing for dbmt language#6.3 Trying to see dbml data representation for when embedding dbmt as comments|Trying to see dbml data representation for when embedding dbmt as comments]] ^spawn-invst-d0d14e

2025-08-15 Wk 33 Fri - 12:31

Filed [#63](https://github.com/Vanderhoof/PyDBML/issues/63) for the unexpected comment preservation behavior with columns.

2025-08-15 Wk 33 Fri - 12:43

We need a more reliable way to preserve dbmt content. Let's keep it simple, don't just comment out lines, generate a comment at the start of the file for a JSON containing the additional dbmt data. Any objects, and what they're associated with, is just written directly there. So we don't need dbml's comment preservation function at all.

2025-08-15 Wk 33 Fri - 14:01

Okay! We have types to distinguish all possible content that is in the [dbmt spec](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md) in `commands.py`.  We use conventions like `AbstCommand` to mean an abstract class since we do not expect to have any concrete instances of `AbstCommand`, only non-abstract derivates.

This is one way to model the types with inheritence, Command -> {Dot/Perc Command} -> Individual commands. We could have also used interfaces/traits and assigned to classes a trait of being a command, or of being a dot command.

For error types, we expect certain commands to have unique errors. A principle we want to follow is that a user can always expect to be able to handle all possible error types declared. So an error that can only occur for `%signal` should not be available in the type for a `%virtual` command for example.  

For this, we (optionally) associate the command type in the error itself, so that the user knows which specific error type to cast it to and handle all possible errors for that command. For shared error types across commands, we have them inherit enum  `ParseErrorType`.  The associated command type in that case would be `None`, since it applies to all.

Yes I guess we can do python enum inheritence...

Now we just need to parse all lines.

2025-08-15 Wk 33 Fri - 14:35

Spawn [[001 Create Python and Rust parsing for dbmt language#5.6 Parse a path string in python|Parse a path string in python]] ^spawn-howto-30bf4d

2025-08-15 Wk 33 Fri - 15:10

A `C-like Token` is identical to a Python Identifier and can be checked with [str.isidentifier](https://www.w3schools.com/python/ref_string_isidentifier.asp).

`Identifier` is a much better name... Let's rename it in the spec.

2025-08-15 Wk 33 Fri - 17:22

It also is `Word(alphas + "_", alphanums + "_")` from this pyparsing [C parser example](https://github.com/pyparsing/pyparsing/blob/master/examples/oc.py). The second argument to `Word` is what is allowed after the first character.

2025-08-15 Wk 33 Fri - 16:28

Spawn [[001 Create Python and Rust parsing for dbmt language#5.7 Python parse enum from string with identical name to identifier|Python parse enum from string with identical name to identifier]] ^spawn-howto-32f6a4

2025-08-15 Wk 33 Fri - 17:00

Spawn [[001 Create Python and Rust parsing for dbmt language#5.8 Use TypeVar variables to create generic functions in python|Use TypeVar variables to create generic functions in python]] ^spawn-howto-862a5c

2025-08-15 Wk 33 Fri - 18:28

Seems there's a contradiction in the [dbmt spec](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md) after we introduced [Signal Trigger Type priority queue](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#343-signal-trigger-type-priority-queue).

In [Signals](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#34-signals), it says:

(Clause 1)

> The `{trigger_type}=[{trigger}, ...]` argument itself may be used one or more times, up to the number of possible trigger types. Repetition of trigger types is not allowed and would flag an error.

But in [Signal Trigger Type priority queue](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#343-signal-trigger-type-priority-queue), it says:

(Clause 2)

> For non-notify signals, there must exist zero or one [subscriber](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#48-signal-subscribing) per [signal trigger type](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#341-signal-trigger-types) for a given set of [triggers](https://github.com/LanHikari22/dbmint/blob/main/docs/dbmt%20specification%20v1.0.0t.md#33-triggers).


(Clause 1) would allow multiple trigger type lists on a single line, this means there can be more than one subscriber for a given set of triggers and (Clause  2) forbids this.

No known use case for (Clause 1) exists as of this writing, so eliminating it in favor of (Clause 2).

This means we're simplifying `%signal` syntax to only be one list a line.

```diff
-%signal {symbol} {{trigger_type}=[{trigger}, ...] ...}
+%signal {symbol} {trigger_type}=[{trigger}, ...]
```

2025-08-15 Wk 33 Fri - 20:17

We made it so that `AbstPercCommand` tracks its associated item:

```python
class AbstPercCommand(AbstCommand):
    associated_item: AssociatedItem

    def get_command_syntax_type(self) -> CommandSyntaxType:
        return CommandSyntaxType.PERC
```

But this `associated_item` information cannot be known by a line parser that only knows the current line. It might be better to track this information separately from percent commands.

2025-08-15 Wk 33 Fri - 20:23

We kept using `pp.Word('virtual')` but this is wrong. This takes characters as input, and NOT an exact string match. I wanted to do this so that it auto-converts into a `ParserElement` for `pyparsing`.

2025-08-22 Wk 34 Fri - 10:55

We're creating a new repository for `dbmt-py`. Let's do this properly and apply what we learned from obsidian-export: 

[7.1 Setting up rustfmt and clippy in CI](https://github.com/LanHikari22/lan-setup-notes/blob/webview/lan/topics/tooling/obsidian/tasks/2025/004%20Fix%20obsidian%20export%20to%20support%20internal%20links.md#71-setting-up-rustfmt-and-clippy-in-ci)
[7.3 Including pre-commit hooks in my repositories](https://github.com/LanHikari22/lan-setup-notes/blob/webview/lan/topics/tooling/obsidian/tasks/2025/004%20Fix%20obsidian%20export%20to%20support%20internal%20links.md#73-including-pre-commit-hooks-in-my-repositories)

Spawn [[001 Create Python and Rust parsing for dbmt language#3.6 Add python pre-commit hooks to the project for style and correctness|Add python pre-commit hooks to the project for style and correctness]] ^spawn-task-11080d


# 3 Tasks

## 3.1 Create a python3 library project template

- [x] 

From [[#^spawn-task-a2863a]] in [[#2 Journal]]

2025-08-14 Wk 33 Thu - 16:27

2025-08-14 Wk 33 Thu - 16:40

Let's use [checkpipe](https://github.com/LanHikari22/checkpipe) as template starter.

Let's also bring some other things from our [dbmint python app](https://github.com/LanHikari22/dbmint/tree/main/app/py): `install.py` -> `install_deps.py` and `mypy.py`.

Modify out the `$project_dir/app/py/` details since this is standalone. 

Also `project_dir="$script_dir/../.."` needs to change to just `$script_dir` since we're not an embedded app. The git root should be where these scripts are.

2025-08-14 Wk 33 Thu - 17:57

We should also include unit tests and integration tests into the template

From [realpython pytest python testing blog](https://realpython.com/pytest-python-testing/), 

Mentions [Arrest-Act-Assert](https://deviq.com/testing/arrange-act-assert).

2025-08-14 Wk 33 Thu - 18:18

Spawn [[001 Create Python and Rust parsing for dbmt language#6.2 Looking at how PyDBML tests their project|Looking at how PyDBML tests their project]] ^spawn-invst-2e2ab9

2025-08-14 Wk 33 Thu - 18:39

We added a `test.sh` that runs `python3 -m unittest` to run all the tests under `tests/` as well as instructions for this in the `README.md`.

2025-08-14 Wk 33 Thu - 19:00

You can find the template python3 library project [here](https://github.com/LanHikari22/lan-exp-scripts/tree/main/templates/2025/topics/py3/persistant/001-lan-library/library).
# 4 Issues

# 5 HowTos

# 6 Investigations

# 7 Ideas

# 8 Side Notes
# 9 External Links

# 10 References