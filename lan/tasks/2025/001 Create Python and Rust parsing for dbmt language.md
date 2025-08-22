
#external

# 1 Objective

Language features specified in [[000 Designing new language dbmt]] are implemented in writers/readers for both Rust and Python.

# 2 Journal

2025-08-06 Wk 32 Wed - 05:08

So there is a project specifically for parsing dbml files in python: [gh Vanderhoof/PyDBML](https://github.com/Vanderhoof/PyDBML).

Correspondingly, there is one for Rust dbml: [gh apskhem/dbml-rs](https://github.com/apskhem/dbml-rs). 

We should follow similarly. Let's fork both of those projects and instead update dbml -> dbmt. The language itself is a separate module from the library that uses it which is [dbmint](https://github.com/LanHikari22/dbmint).

Thinking about whether to fork or just start a project that uses those projects internally. This [opensource.stackexchange post](https://opensource.stackexchange.com/questions/1934/rules-guidelines-about-forking-a-project-vs-creating-a-new-one) discusses some of benefits of forking, even if  no PR is planned or the project diverges.

I think in my case, `.dbmt` has different features and approaches, I will maintain the underlying dbml packages as dependencies. 

2025-08-06 Wk 32 Wed - 08:49

~~Let's fork the projects after all. From this [[#4.1 assertEqual on dbml tables fail even though they are copied from print|issue]] we can see there would be dependency problems if our dbmt-py library depended on pydbml since we're being fixed at an older version by the sqlite converting library. And the language really is a fork of dbml goin in another direction.~~

We're going to make them standalone projects with dependencies. For the dependency issue, we could fork the 

Spawn [[#3.2 Implement dbmt-py]]. ^spawn-task-060600


2025-08-06 Wk 32 Wed - 16:32

They added [Arbitrary Properties](https://github.com/Vanderhoof/PyDBML/blob/master/docs/properties.md#arbitrary-properties) to the dbml language. I might be able to use this for some feature I planned for [[000 Designing new language dbmt|dbmt]].

# 3 Tasks


## 3.1 Go through reading and writing dbml in rust and python 

- [ ] 

### 3.1.1 Reading and writing in Python

2025-08-06 Wk 32 Wed - 02:58

```sh
./docker_build.sh && MOUNT="./mnt" ./docker_run.sh rust gen-lib a.dbml
```

2025-08-06 Wk 32 Wed - 03:59

It seems in the case that multiple files like `main.py` and `dbml.py` include the same unit test class name like `class Module1UnitTests(unittest.TestCase)`, only the main one runs and gives OK even though I have made something fail in the `dbml.py` one. Might be best to give them unique names, then this issue doesn't happen.

2025-08-06 Wk 32 Wed - 04:32

Spawn [[#4.1 assertEqual on dbml tables fail even though they are copied from print]]. ^spawn-issue-060433

2025-08-06 Wk 32 Wed - 16:58

Now we're able to read dbml in python. For writing, according to the [creating schema docs](https://github.com/Vanderhoof/PyDBML/blob/master/docs/creating_schema.md), we just have to create a database object and render via `db.dbml`. There's also `db.sql`. 

The output isn't identical to what I have written in style, but it should be in content. I've written it to `mnt/ex000.autogen.dbml`.

### 3.1.2 Reading in Rust

2025-08-06 Wk 32 Wed - 17:21

It seems so far for rust we only have a reader? See [gh apskhem/dbml-rs](https://github.com/apskhem/dbml-rs).

For our purposes right now, we're mostly interested in reading the files. We could use python for writing.


```sh
# in /home/lan/src/cloned/gh/apskhem/dbml-rs
ls
Cargo.toml  LICENSE-APACHE  LICENSE-MIT  README.md  rustfmt.toml  src  tests
```

I guess this is how we add two different licenses? This project said this can use either...

2025-08-06 Wk 32 Wed - 17:43

The README.md example doesn't build and is missing `.unwrap()`s. Also that example is not very clear. We should open a PR to add come clear usage examples similar to what we did with python above over an example. 

Spawn [[#3.6 Open a PR for dbml-rs to add better examples and fix README.md broken example]]. ^spawn-task-061745

### 3.1.3 Later

## 3.2 Implement dbmt-py

- [ ] 

From [[#^spawn-task-060600]].

2025-08-06 Wk 32 Wed - 06:01

2025-08-14 Wk 33 Thu - 15:52

Let's create the project!

2025-08-14 Wk 33 Thu - 16:23

We need a python3 library project template. Something similar to [checkpipe](https://github.com/LanHikari22/checkpipe). with a `requirements.txt` and `pyproject.toml`. 

Spawn [[#3.7 Create a python3 library project template]] ^spawn-task-a2863a

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

Spawn [[#6.3 Trying to see dbml data representation for when embedding dbmt as comments]] ^spawn-invst-d0d14e

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

Spawn [[#5.6 Parse a path string in python]] ^spawn-howto-30bf4d

2025-08-15 Wk 33 Fri - 15:10

A `C-like Token` is identical to a Python Identifier and can be checked with [str.isidentifier](https://www.w3schools.com/python/ref_string_isidentifier.asp).

`Identifier` is a much better name... Let's rename it in the spec.

2025-08-15 Wk 33 Fri - 17:22

It also is `Word(alphas + "_", alphanums + "_")` from this pyparsing [C parser example](https://github.com/pyparsing/pyparsing/blob/master/examples/oc.py). The second argument to `Word` is what is allowed after the first character.

2025-08-15 Wk 33 Fri - 16:28

Spawn [[#5.7 Python parse enum from string with identical name to identifier]] ^spawn-howto-32f6a4

2025-08-15 Wk 33 Fri - 17:00

Spawn [[#5.8 Use TypeVar variables to create generic functions in python]] ^spawn-howto-862a5c

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

Spawn [[#3.8 Add python pre-commit hooks to the project for style and correctness]] ^spawn-task-11080d


### 3.2.1 Pend

## 3.3 Add Docker build option to provide a shell for troubleshooting

- [x] 

From [[#^spawn-task-060606]] in [[#4.1 assertEqual on dbml tables fail even though they are copied from print]]

2025-08-06 Wk 32 Wed - 06:15

Similar to the [delta-box box000_blank_system](https://github.com/delta-domain-rnd/delta-box/tree/main/box/box000_blank_system) system, we want to be able to just start a shell for testing purposes rather than an application.

Let's use [docker_root_bash.sh](https://github.com/delta-domain-rnd/delta-box/blob/main/box/box000_blank_system/docker_root_bash.sh).

```sh
# in /home/lan/src/cloned/gh/LanHikari22/dbmint
cp ~/src/cloned/gh/delta-domain-rnd/delta-box/box/box000_blank_system/docker_root_bash.sh .
```

We need to update it with the app name

```sh
./edit_app_name.sh                                                                                 
# out
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/docker_root_bash.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/app/py/mypy.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/app/py/run.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/docker_build.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/docker_clean.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/docker_publish.sh
OK /home/lan/src/cloned/gh/LanHikari22/dbmint/docker_run.sh
```

Hardcode the tag environment variable to latest

```sh
# in docker_root_bash.sh
export TAG=latest
```

The remaining issue is that the [entrypoint](https://github.com/delta-domain-rnd/delta-box/blob/main/box/box000_blank_system/boot/entrypoint.sh) for box000 stays on indefinitely while ours just executes the application and exists. So we need an option to change this.

Similar to [box000 docker_build.sh](https://github.com/delta-domain-rnd/delta-box/blob/main/box/box000_blank_system/docker_build.sh),

We're adding an environment option `USE_SHELL`:

```sh
docker build \
  -t lan22h/$app_name:latest \
  --build-arg USE_SHELL=$USE_SHELL \
  .
```

2025-08-06 Wk 32 Wed - 06:37

Now we would like to export this environment variable within the image itself so `entrypoint.sh` can use it

Also, by default it's disabled.

```sh
# in Dockerfile
ARG USE_SHELL=0
```

2025-08-06 Wk 32 Wed - 06:52

I wanted to configure bash to have an export, but we need this to be system wide available. Just create a file based on this option. 

2025-08-06 Wk 32 Wed - 07:10

```Dockerfile
ARG USE_SHELL
RUN /bin/sh -c '\
  if [ $USE_SHELL ]; then \
    echo "yes" > /var/use_shell; \
  else \
    echo "no" > /var/use_shell; \
  fi \
'
```

For testing, I set `boot/entrypoint.sh` to

```sh
#!/bin/sh

echo "Hello world!"
cat /var/use_shell
```

```sh
./docker_build.sh && MOUNT="./mnt" ./docker_run.sh

# out (relevant)
Hello world!
no
```

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_run.sh

# out (relevant)
Hello world!
yes
```

2025-08-06 Wk 32 Wed - 07:16

Our `boot/entrypoint.sh` now is

```sh
#!/bin/sh

if [ $(cat /var/use_shell) == "yes" ]; then
  # Stay indefinitely
  tail -f /dev/null
else
  # Just run the app
  . /opt/venv/bin/activate && \
    PYTHONPATH=/app/src python3 -m dbmint.main $@
fi
```

2025-08-06 Wk 32 Wed - 07:18

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_root_bash.sh

# out (relevant)
OCI runtime exec failed: exec failed: unable to start container process: exec: "bash": executable file not found in $PATH: unknown
Bye!

```

Right we don't have bash for alpine by default. Let's just try running `/bin/sh`. Get [box000 docker_sh.sh](https://github.com/delta-domain-rnd/delta-box/blob/main/box/box000_blank_system/docker_sh.sh) instead.

```sh
# in /home/lan/src/cloned/gh/LanHikari22/dbmint
rm docker_root_bash.sh
cp ~/src/cloned/gh/delta-domain-rnd/delta-box/box/box000_blank_system/docker_root_sh.sh . 
./edit_app_name.sh

# in docker_root_sh.sh
export TAG=latest
```

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_root_sh.sh
```

We're in!

```sh
/ # apk add bash
(1/1) Installing bash (5.2.37-r0)
Executing bash-5.2.37-r0.post-install
Executing busybox-1.37.0-r18.trigger
OK: 221 MiB in 53 packages
/ # bash
470031744a8f:/# 
```

We can use `apk` to add new packages for temporary testing or anything we want. This container is killed when we exist with `^D`. 

## 3.4 Open a PR for  DBML_SQLite and bump version of pydbml

- [ ] Watch

From [[#^spawn-task-060923]].

This is for [PR #9](https://github.com/dvanderweele/DBML_SQLite/pull/9).

2025-08-06 Wk 32 Wed - 09:23

Associated issue is [issue #8](https://github.com/dvanderweele/DBML_SQLite/issues/8) for [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/) to bump version.

Let's fork [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/).

```sh
mkdir -p ~/src/cloned/gh/LanHikari22/forked/dvanderweele
cd ~/src/cloned/gh/LanHikari22/forked/dvanderweele
git clone git@github.com:LanHikari22/DBML_SQLite.git
cd DBML_SQLite
```

Let's also correct our convention of `forked/` for other projects. We should always include the users or organizations that own the project

```
mkdir -p ~/src/cloned/gh/LanHikari22/forked/apache/
mv ~/src/cloned/gh/LanHikari22/forked/arrow-rs ~/src/cloned/gh/LanHikari22/forked/apache/arrow-rs
mkdir -p ~/src/cloned/gh/LanHikari22/forked/ron-rs
mv ~/src/cloned/gh/LanHikari22/forked/ron  ~/src/cloned/gh/LanHikari22/forked/ron-rs
```

Let's create a fix branch for bumping the version.

```sh
git checkout -b fix-bump-pydbml-v1.2.0
```

```diff
# in requirements.txt
-pydbml==0.3.4
+pydbml==1.2.0
```

We should run the tests recommended in the [README.md](https://github.com/dvanderweele/DBML_SQLite/?tab=readme-ov-file#testing-and-coverage),

We need to install [poetry](https://python-poetry.org/) first

```sh
python3 -m pip install poetry
```

```sh
poetry run pytest

# out (exit=1)
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.
Creating virtualenv dbml-sqlite-Sv97MqvY-py3.13 in /home/lan/.cache/pypoetry/virtualenvs
Command not found: pytest
```

```sh
poetry run coverage run --source dbml_sqlite -m pytest

# out (exit=1)
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.
Command not found: coverage
```

Let's install pytest and install the fork

```sh
python3 -m pip install pytest
```

2025-08-06 Wk 32 Wed - 09:56

[[#5.2 Installing a python project with a pyproject.toml|HowTo]]. ^spawn-howto-060957

2025-08-06 Wk 32 Wed - 10:14

```sh
python3 -m pip install -e .
```

```sh
poetry run pytest

# out (relevant)
def test_process_column():

>       assert processColumn(c2, 'full') == '  c2 INTEGER PRIMARY KEY'

tests/test_dbml_sqlite.py:93: 

>               if column.autoinc:
E               AttributeError: 'MockColumn' object has no attribute 'autoinc'

dbml_sqlite/core.py:216: AttributeError
```
^pr-4-tests-failing

This issue occurs even on pydbml v0.3.4.

I've been modifying requirement.txt version, but not pyproject.toml... Either way will stay v0.3.4 for now to see if we can get the tests to run.

2025-08-06 Wk 32 Wed - 10:35

```sh
git blame dbml_sqlite/core.py

# out (relevant)
6e0c970e (Philipp Hörist   2023-06-14 20:32:37 +0200 216)             if column.autoinc:
6e0c970e (Philipp Hörist   2023-06-14 20:32:37 +0200 217)                 segments.append(' AUTOINCREMENT')
```

Seems to be due to a latest PR change?

```
commit 24a437eb08674d4be6ede97f7bdcbae86a28c562
Merge: 8fdc78b 6e0c970
Author: Dave VanderWeele <weele.me@gmail.com>
Date:   Mon Nov 25 16:52:02 2024 -0600

    Merge pull request #4 from lovetox/main
    
    Add support for AUTOINCREMENT

commit 6e0c970eb2d8cd1d1696dd61dc0832317055bd3a
Author: Philipp Hörist <philipp@hoerist.com>
Date:   Wed Jun 14 20:32:37 2023 +0200

    Add support for AUTOINCREMENT
```

2025-08-06 Wk 32 Wed - 10:41

This is PR [#4](https://github.com/dvanderweele/DBML_SQLite/pull/4). The author [states](https://github.com/dvanderweele/DBML_SQLite/pull/4#issuecomment-2499196571) there that they have abandoned the project and have not tested this PR and may turn it read-only.

Let's go to the commit before the PR and see if tests run ok

```sh
git checkout -b 8fdc78bb23f2ffe12b9be583d1792d25032aae5f
python3 -m pip install -e .
poetry run pytest

# out (relevant)
tests/test_dbml_sqlite.py ...........                                                                                                                                                  [ 91%]
tests/test_terminal.py .                                                                                                                                                               [100%]

12 passed in 0.46s
```

They work fine. 

What about after bumping versions?

```diff
# in pyproject.toml
-pydbml = "^0.3.4"
+pydbml = "^1.2.0"
 
# in requirements.txt
-pydbml==0.3.4
+pydbml==1.2.0
```

```sh
python3 -m pip install -e .
poetry run pytest


# out (relevant)
tests/test_dbml_sqlite.py .F......F.F 
tests/test_terminal.py F 

>       for j, ref in enumerate(table.refs):
                                ^^^^^^^^^^
E       AttributeError: 'Table' object has no attribute 'refs'

dbml_sqlite/core.py:164: AttributeError
```

2025-08-06 Wk 32 Wed - 10:55

[[#6.1 Invst what happened to table.refs for pydbml|Invst]]. ^spawn-invst-061055

2025-08-06 Wk 32 Wed - 12:01

There is a [get_refs](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/table.py#L115) function that can still be used. Let's change refs to that.

2025-08-06 Wk 32 Wed - 12:32

```diff
# in dbml_sqlite/core.py

@@ -186,7 +186,8 @@ def processRef(ref, join=True):
-    segments.append(f'{ref.col.name}) REFERENCES {ref.ref_table.name}({ref.ref_col.name})')
+    segments.append(f'{ref.col1[0].name}) REFERENCES {ref.col2[0].table.name}({ref.col2[0].name})')
```

2025-08-06 Wk 32 Wed - 12:57

Luckily the PR [#4](https://github.com/dvanderweele/DBML_SQLite/pull/4) issue [[#^pr-4-tests-failing]] wasn't so bad. Just had to add an extra field to the mock class to get the tests to run.

Let's use the same convention [here](https://github.com/dvanderweele/DBML_SQLite/pull/4#issue-1757442972) where we write in the PR `Fixes #N` for the issue ID N.

2025-08-06 Wk 32 Wed - 13:08

```sh
remote: Create a pull request for 'fix-bump-pydbml-v1.2.0' on GitHub by visiting:
remote:      https://github.com/LanHikari22/DBML_SQLite/pull/new/fix-bump-pydbml-v1.2.0
```

Let's do it!

See [#9](https://github.com/dvanderweele/DBML_SQLite/pull/9).

2025-08-06 Wk 32 Wed - 13:37

This might be merged in. There are open PRs on this project that fix various issues. Let's try to merge them ourselves into the fork master branch. [[#3.5 Merge currently open PRs into fork main for DBML_SQLite|Task]]. ^spawn-task-061339

### 3.4.1 Watch1

## 3.5 Merge currently open PRs into fork main for DBML_SQLite

- [x] 

From [[#^spawn-task-061339]].

2025-08-06 Wk 32 Wed - 13:39

2025-08-06 Wk 32 Wed - 13:57

This [stackoverflow answer](https://stackoverflow.com/questions/1425892/how-do-you-merge-two-git-repositories#10548919 ) could help us merge those PRs into our fork. 

Let's get the PRs,

We want [#6](https://github.com/dvanderweele/DBML_SQLite/pull/6) and [#7](https://github.com/dvanderweele/DBML_SQLite/pull/7).

Let's merge our PR in our main,

```sh
# in /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite
# in main branch
git merge fix-bump-pydbml-v1.2.0
```

Let's start with [#6](https://github.com/dvanderweele/DBML_SQLite/pull/6),

```sh
mkdir -p ~/src/cloned/gh/dvanderweele/forks/KoStyle/
cd ~/src/cloned/gh/dvanderweele/forks/KoStyle/
git clone git@github.com:KoStyle/DBML_SQLite.git
cd /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite

# in main branch
git remote add sibling ~/src/cloned/gh/dvanderweele/forks/KoStyle/DBML_SQLite
git fetch sibling
git merge sibling/main
git remote remove sibling
```

We do pass all tests via

```sh
python3 -m pip install -e .
poetry run pytest
```

Repeating for [#7](https://github.com/dvanderweele/DBML_SQLite/pull/7),

```sh
mkdir -p ~/src/cloned/gh/dvanderweele/forks/TheDimensionofEternity/
cd ~/src/cloned/gh/dvanderweele/forks/TheDimensionofEternity/
git clone git@github.com:TheDimensionofEternity/DBML_SQLite.git
cd /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite

# in main branch
git remote add sibling ~/src/cloned/gh/dvanderweele/forks/TheDimensionofEternity/DBML_SQLite
git fetch sibling
git merge sibling/main
git remote remove sibling

python3 -m pip install -e .
poetry run pytest
```

All tests pass.

2025-08-06 Wk 32 Wed - 14:19

I guess I should've just mentioned #6 and #7. Github is able to reference them on my fork.

![[Pasted image 20250806141949.png]]

Name is a bit weird, but it's alright. It'd take force pushing to change this, and it's synced with three sources. 

## 3.6 Open a PR for dbml-rs to add better examples and fix README.md broken example

- [ ] 

From [[#^spawn-task-061745]] in [[#3.1 Go through reading and writing dbml in rust and python]]

### 3.6.1 Pend

## 3.7 Create a python3 library project template

- [x] 

From [[#^spawn-task-a2863a]] in [[#3.2 Implement dbmt-py]].

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

Spawn [[#6.2 Looking at how PyDBML tests their project]] ^spawn-invst-2e2ab9

2025-08-14 Wk 33 Thu - 18:39

We added a `test.sh` that runs `python3 -m unittest` to run all the tests under `tests/` as well as instructions for this in the `README.md`.

2025-08-14 Wk 33 Thu - 19:00

You can find the template python3 library project [here](https://github.com/LanHikari22/lan-exp-scripts/tree/main/templates/2025/topics/py3/persistant/001-lan-library/library).

## 3.8 Add python pre-commit hooks to the project for style and correctness

- [ ] 

From [[#^spawn-task-11080d]] in [[#3.2 Implement dbmt-py]]

2025-08-22 Wk 34 Fri - 11:13

2025-08-22 Wk 34 Fri - 11:34

Added instructions in `CONTRIBUTING.md` for setting up the pre-commit hooks.

From [pre-commit](https://pre-commit.com/),

```sh
pre-commit sample-config > .pre-commit-config.yaml
```

We want [psf/black](https://github.com/psf/black) formatting in python. We can also [setup an icon](https://github.com/psf/black?tab=readme-ov-file#show-your-style) in our repo according to black:

```
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
```

They had another option also for README.rst which is innteresting. I just use README.md

We added it as

```html
  <a href="https://github.com/psf/black">
    <img src="https://img.shields.io/badge/code%20style-black-000000.svg" alt="Style: Black"/>
  </a>
```

For consistency with the other icons and centering.

(update)
Add `Black` as a hook:

```yaml
repos:
  # Using this mirror lets us use mypyc-compiled black, which is about 2x faster
  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 25.1.0
    hooks:
      - id: black
        # It is recommended to specify the latest version of Python
        # supported by your project here, or alternatively use
        # pre-commit's default_language_version, see
        # https://pre-commit.com/#top_level-default_language_version
        language_version: python3.11
```

<details>
<summary>errata</summary>

2025-08-22 Wk 34 Fri - 15:08

I mentioned

```yaml
-   repo: https://github.com/psf/black
    rev: 25.1.0
    hooks:
    -   id: black
```

but in [Version control integration](https://github.com/psf/black/blob/main/docs/integrations/source_version_control.md#version-control-integration) they recommended

```yaml
repos:
  # Using this mirror lets us use mypyc-compiled black, which is about 2x faster
  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 25.1.0
    hooks:
      - id: black
        # It is recommended to specify the latest version of Python
        # supported by your project here, or alternatively use
        # pre-commit's default_language_version, see
        # https://pre-commit.com/#top_level-default_language_version
        language_version: python3.11
```





</details>

(/update)

2025-08-22 Wk 34 Fri - 12:17

Spawn [[#3.9 File issue for non-recommended black hooks in pre-commit]] ^spawn-task-9bdc4c

2025-08-22 Wk 34 Fri - 15:07

That issues was not necessary, they do not want the documentation to be up to date with examples from other sources it seems.

2025-08-22 Wk 34 Fri - 15:27

[mirrors-mypy](https://github.com/pre-commit/mirrors-mypy) has a recommended hook. Let's adapt it as:

```
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: 'v1.17.1'
    hooks:
      - id: mypy
        args: [--strict, --ignore-missing-imports]
```

Notice we also changed where the dashes are from `-   repo` to `  - repo` for example. We'll also have to watch out with this since mypy isn't going to scan third-party dependencies just with this. They mention because it's put in its own isolated virtualenv.

Let's also use a linter. There's [ruff](https://github.com/astral-sh/ruff) which seems very fast. They [also](https://github.com/zoni/obsidian-export/pull/373/commits/9841ecff2283585518b0374e24a67217af778959) recommend uv since it's faster than pip... Maybe we can do this too in the project.

Here is their [hook](https://github.com/astral-sh/ruff-pre-commit):

```
repos:
- repo: https://github.com/astral-sh/ruff-pre-commit
  # Ruff version.
  rev: v0.12.10
  hooks:
    # Run the linter.
    - id: ruff-check
      args: [ --fix ]
    # Run the formatter.
    - id: ruff-format
```

They have a formatter too... Let's be more integrated and use ruff then.

Here is their [icon](https://github.com/astral-sh/ruff?tab=readme-ov-file#show-your-support):

```html
<a href="https://github.com/astral-sh/ruff"><img src="https://img.shields.io/endpoint?url=https://raw.githubusercontent.com/astral-sh/ruff/main/assets/badge/v2.json" alt="Ruff" style="max-width:100%;"></a>
```



### 3.8.1 Pend


## 3.9 File issue for non-recommended black hooks in pre-commit

- [x] 

From [[#^spawn-task-9bdc4c]] in [[#3.8 Add python pre-commit hooks to the project for style and correctness]]

2025-08-22 Wk 34 Fri - 12:17

In Black's [.pre-commit-hooks.yaml](https://github.com/psf/black/blob/main/.pre-commit-hooks.yaml) they write:

> Note that we recommend using https://github.com/psf/black-pre-commit-mirror instead
 > This will work about 2x as fast as using the hooks in this repository

but yet [pre-commit](https://pre-commit.com/#2-add-a-pre-commit-configuration) uses the non-recommended configuration. 

2025-08-22 Wk 34 Fri - 12:50

2025-08-22 Wk 34 Fri - 14:23

Filed [#3526](https://github.com/pre-commit/pre-commit/issues/3526)

# 4 Issues

## 4.1 assertEqual on dbml tables fail even though they are copied from print

- [x] 

From [[#^spawn-issue-060433]] in [[#3.1 Go through reading and writing dbml in rust and python]]

2025-08-06 Wk 32 Wed - 04:34

```sh
./docker_build.sh && MOUNT="./mnt" ./docker_run.sh unittest DbmlUnitTests.test_example_parsing_dbml
```

```python
print(dbml_data.tables)

# out
[Table('Users', [Column('id', 'integer', pk=True), Column('username', 'varchar'), Column('role', 'varchar'), Column('created_at', 'varchar')]), Table('Posts', [Column('id', 'integer', pk=True), Column('title', 'varchar'), Column('body', 'varchar', note=Note('Content of the post')), Column('status', 'varchar'), Column('created_at', 'varchar')]), Table('User_Owns_Posts', [Column('id', 'integer', pk=True), Column('from_user_id', 'integer'), Column('to_post_id', 'integer')])]
```

And yet this fails

```python
expected_tables = [
	Table('Users', [
		Column('id', 'integer', pk=True),
		Column('username', 'varchar'),
		Column('role', 'varchar'),
		Column('created_at', 'varchar'),
	]),

	Table('Posts', [
		Column('id', 'integer', pk=True),
		Column('title', 'varchar'), 
		Column('body', 'varchar', note=Note('Content of the post')), 
		Column('status', 'varchar'), 
		Column('created_at', 'varchar'), 
	]),

	Table('User_Owns_Posts', [
		Column('id', 'integer', pk=True), 
		Column('from_user_id', 'integer'), 
		Column('to_post_id', 'integer'),
	])
]

self.assertEqual(dbml_data.tables, expected_tables)
```
^ex000-as-expected-1

2025-08-06 Wk 32 Wed - 04:43

We can't hash the lists directly

```
TypeError: unhashable type: 'list'
```

```python
print(hash(str(dbml_data.tables)), hash(str(expected_tables)))

# out
-7912868938497380503 8786281224489275648
```

```python
print(expected_tables)

# out
[Table('Users', [], alias=[Column('id', 'integer', pk=True), Column('username', 'varchar'), Column('role', 'varchar'), Column('created_at', 'varchar')]), Table('Posts', [], alias=[Column('id', 'integer', pk=True), Column('title', 'varchar'), Column('body', 'varchar', note=Note('Content of the post')), Column('status', 'varchar'), Column('created_at', 'varchar')]), Table('User_Owns_Posts', [], alias=[Column('id', 'integer', pk=True), Column('from_user_id', 'integer'), Column('to_post_id', 'integer')])]
```

The table is set to alias instead...

Seems we need to use [add_column](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/table.py#L70) or a [columns](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/table.py#L30) keyword argument.

```python
expected_tables = [
	Table('Users', columns=[
		Column('id', 'integer', pk=True),
		Column('username', 'varchar'),
		Column('role', 'varchar'),
		Column('created_at', 'varchar'),
	]),

	Table('Posts', columns=[
		Column('id', 'integer', pk=True),
		Column('title', 'varchar'), 
		Column('body', 'varchar', note=Note('Content of the post')), 
		Column('status', 'varchar'), 
		Column('created_at', 'varchar'), 
	]),

	Table('User_Owns_Posts', columns=[
		Column('id', 'integer', pk=True), 
		Column('from_user_id', 'integer'), 
		Column('to_post_id', 'integer'),
	])
]
```

2025-08-06 Wk 32 Wed - 05:04

We don't have that columns keyword argument. Do we not have the last version?

```
TypeError: Table.__init__() got an unexpected keyword argument 'columns'
```

```sh
python3 -m pip install pydbml 

# out
Requirement already satisfied: pydbml in /home/lan/miniconda3/lib/python3.13/site-packages (0.3.4)
Requirement already satisfied: pyparsing>=2.4.7 in /home/lan/miniconda3/lib/python3.13/site-packages (from pydbml) (3.2.3)
```

2025-08-06 Wk 32 Wed - 06:03

Spawn [[#3.3 Add Docker build option to provide a shell for troubleshooting]]. ^spawn-task-060606

2025-08-06 Wk 32 Wed - 07:30

Now that we can spawn a shell, let's examine the versions of the libraries we have.

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_root_sh.sh

# in shell
/opt/venv/bin/pip3 show pydbml

# out
Name: pydbml
Version: 0.3.4
Summary: DBML syntax parser for Python
Home-page: https://github.com/Vanderhoof/PyDBML
Author: Daniil Minukhin
Author-email: ddddsa@gmail.com
License: MIT
Location: /opt/venv/lib/python3.12/site-packages
Requires: pyparsing
Required-by: dbml-sqlite
```

We can see that the version from the code we checked is all the way up to [v.1.2.0](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/setup.py#L19).

We can see it's being set to [v0.3.4](https://github.com/dvanderweele/DBML_SQLite/blob/10486060a49123ed5d62b22961f96038e9bb09e1/requirements.txt#L23) explicitly by [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/).

Let's add a requirement to our app in `app/py/requirements.txt`

```
pydbml
```

and see if it fetches the latest version.

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_root_sh.sh

# in shell
/opt/venv/bin/pip3 show pydbml

# out (relevant)
Version: 0.3.4
```

No. Let's fix it to v1.2.0 in `app/py/requirements.txt`,

```
pydbml==1.2.0
```

Right, we get a build error.

```
9.062     The user requested pydbml==1.2.0
9.062     dbml-sqlite 0.1.0 depends on pydbml<0.4.0 and >=0.3.4
```

I created an [issue #8](https://github.com/dvanderweele/DBML_SQLite/issues/8) for [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/) to bump version.

Alright let's just stick with `v0.3.4` for now  and use the `add_column`. We're just gonna specify `pydbml` in the requirements since we do use it directly.

2025-08-06 Wk 32 Wed - 08:34

```python
mut_expected_tables: List[Table] = []

tmp_table0 = Table('Users')
tmp_table0.add_column(Column('id', 'integer', pk=True))
tmp_table0.add_column(Column('username', 'varchar'))
tmp_table0.add_column(Column('role', 'varchar'))
tmp_table0.add_column(Column('created_at', 'varchar'))
mut_expected_tables.append(tmp_table0)

tmp_table1 = Table('Posts')
tmp_table1.add_column(Column('id', 'integer', pk=True))
tmp_table1.add_column(Column('title', 'varchar'))
tmp_table1.add_column(Column('body', 'varchar', note=Note('Content of the post')))
tmp_table1.add_column(Column('status', 'varchar'))
tmp_table1.add_column(Column('created_at', 'varchar'))
mut_expected_tables.append(tmp_table1)

tmp_table2 = Table('User_Owns_Posts')
tmp_table2.add_column(Column('id', 'integer', pk=True))
tmp_table2.add_column(Column('from_user_id', 'integer'))
tmp_table2.add_column(Column('to_post_id', 'integer'))
mut_expected_tables.append(tmp_table2)

print(hash(str(dbml_data.tables)), hash(str(mut_expected_tables)))

# out
7832558014608569827 7832558014608569827
```

Don't expect the hashes to be identical to the last, they're always different.

2025-08-06 Wk 32 Wed - 10:05

We're trying to bump versions with this [[#3.4 Open a PR for DBML_SQLite and bump version of pydbml|PR Task]] to see if that helps the problem here. ^spawn-task-060923

2025-08-06 Wk 32 Wed - 14:31

Okay so let's use my [DBML_SQLite fork](https://github.com/LanHikari22/DBML_SQLite). We need to be able to replace our dbmint requirements.txt of `dbml-sqlite` with my fork. This should be using [v.1.2.0](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/setup.py#L19). [[#5.5 Add python project from github to requirements.txt|HowTo]]. ^spawn-howto-061436

2025-08-06 Wk 32 Wed - 14:42

Let's add a tag with the new version number for my [DBML_SQLite fork](https://github.com/LanHikari22/DBML_SQLite),

[tagging git reference](https://git-scm.com/book/en/v2/Git-Basics-Tagging).

```sh
# in /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite
git tag | cat

# out
v0.1.0
v0.2.0
v0.2.3
v0.2.4
v0.2.5
v0.3.1
v0.3.2
v0.3.3
```

```sh
git tag -a "v0.3.4" -m "Lan's fork"
git push origin v0.3.4
```

You can view the tag [here](https://github.com/LanHikari22/DBML_SQLite/releases/tag/v0.3.4).

Now we should be able to add to requirements.txt,

```
dbml-sqlite @ git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4
```

```sh
./docker_build.sh && MOUNT="./mnt" ./docker_run.sh unittest DbmlUnitTests.test_example_parsing_dbml

# out (relevant)
1.609 ERROR: Cannot find command 'git' - do you have 'git' installed and in your PATH?
```

I guess we have to install git in our image now...

```diff
# in Dockerfile
 # Install necessary packages
- RUN apk update && apk add --no-cache \
+RUN apk update && apk add --no-cache \
   python3 \
   py3-pip \
   gcc \
+  git \
   sqlite
```

2025-08-06 Wk 32 Wed - 14:55

My test seems to pass the first example.

I just need to update the installation instructions in the [DBML_SQLite fork](https://github.com/LanHikari22/DBML_SQLite) README. I also need to make sure the tag is at the latest commit. 

Which, from [this stackoverflow answer](https://stackoverflow.com/a/8044605/6944447), we can do like this:

```sh
git push origin :refs/tags/v0.3.4
git tag -fa v0.3.4
git push origin --tags
git push origin main
```

2025-08-06 Wk 32 Wed - 15:03

```sh
USE_SHELL=1 ./docker_build.sh && MOUNT="./mnt" ./docker_root_sh.sh

# in shell
/opt/venv/bin/pip3 show pydbml

# out (relevant)
Version: 1.2.0
```

Cool, we're on latest pydbml now! So we should be able to create our dbmt-py and use this as dependency just fine.

2025-08-06 Wk 32 Wed - 15:20

We can know use

![[#^ex000-as-expected-1]]

2025-08-06 Wk 32 Wed - 15:38

This is useful!

```sh
poetry add checkpipe
```

on to `pyproject.toml`

2025-08-06 Wk 32 Wed - 16:08

```sh
poetry add git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4

# out
'Tag' object has no attribute 'parents'
```

Not sure what's with this... It seems to have given a different issue before, then I bumped [DBML_SQLite fork](https://github.com/LanHikari22/DBML_SQLite) python requirement to $\ge 3.8$ and it's back to this issue... 

Spawn [[#4.2 poetry add github project says Tag object has no attribute parents]]. ^spawn-issue-061610

2025-08-06 Wk 32 Wed - 16:11

I think the issue is that the third table is supposed to have a reference.

There's a [creating_schema.md](https://github.com/Vanderhoof/PyDBML/blob/master/docs/creating_schema.md) documentation.

2025-08-06 Wk 32 Wed - 16:40

```sh
./docker_build.sh && MOUNT="./mnt" ./docker_run.sh unittest DbmlUnitTests.test_example_parsing_dbml

# out (relevant)
FAIL: test_example_parsing_dbml (dbmint.dbml.DbmlUnitTests.test_example_parsing_dbml)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/app/src/dbmint/dbml.py", line 74, in test_example_parsing_dbml
    self.assertEqual(dbml_data.tables[2], expected_tables[2])
AssertionError: <Table 'public' 'User_Owns_Posts'> != <Table 'public' 'User_Owns_Posts'>
```

How are they not the same? Why are the first objects the same? Is it really because of the references? Those aren't properties here. 

```python
print(table2.__dict__)
print(dbml_data.tables[2].__dict__)

# out 
{'database': <Database>, 'name': 'User_Owns_Posts', 'schema': 'public', 'columns': [<Column 'id', 'integer'>, <Column 'from_user_id', 'integer'>, <Column 'to_post_id', 'integer'>], 'indexes': [], 'alias': None, '_note': Note(''), 'header_color': None, 'comment': None, 'abstract': False, 'properties': {}}
{'database': <Database>, 'name': 'User_Owns_Posts', 'schema': 'public', 'columns': [<Column 'id', 'integer'>, <Column 'from_user_id', 'integer'>, <Column 'to_post_id', 'integer'>], 'indexes': [], 'alias': None, '_note': Note(''), 'header_color': None, 'comment': 'A one-to-many relationship between users and posts', 'abstract': False, 'properties': {}}

```

Oh The table had a comment!

2025-08-06 Wk 32 Wed - 16:46

Okay! The tables are identical now. We need to test for the references too

```python
print(dbml_data.refs)

# out
[<Reference '-', ['from_user_id'], ['id']>, <Reference '-', ['to_post_id'], ['id']>]
```

OK! We're able to confirm all the data in `ex000.dbml`!


## 4.2 poetry add github project says Tag object has no attribute parents

- [ ] 

From [[#^spawn-issue-061610]] in [[#4.1 assertEqual on dbml tables fail even though they are copied from print]]

```sh
poetry add git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4

# out
'Tag' object has no attribute 'parents'
```

2025-08-06 Wk 32 Wed - 16:11

I need this to work so I confirm that the [DBML_SQLite fork](https://github.com/LanHikari22/DBML_SQLite) README is correct...

### 4.2.1 Pend

# 5 HowTos

## 5.1 Disable python formatting on a specific block

- [x] 

2025-08-06 Wk 32 Wed - 03:07

The defaults are not always readable... For example it takes

```python
    rename_commands = (
        SyncCommandsImpl.Rename
        .from_str(dbml_schema_content) 
        | pipe.Of[Result[List[SyncCommandsImpl.Rename], Error]].map(
            lambda res: unwrap_or_print_and_exit_on_app_err(res).extract(
                caller_returns_IO=True)
        )
    )
```

and turns it into 

```python
    rename_commands = SyncCommandsImpl.Rename.from_str(dbml_schema_content) | pipe.Of[
        Result[List[SyncCommandsImpl.Rename], Error]
    ].map(
        lambda res: unwrap_or_print_and_exit_on_app_err(res).extract(
            caller_returns_IO=True
        )
    )
```

I'm using the [Black formatter](https://marketplace.visualstudio.com/items?itemName=ms-python.black-formatter) extension in vscode.

From [this stackoverflow answer](https://stackoverflow.com/a/58584557) which references [black docs](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html#ignoring-sections), the [ignoring section](https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html#ignoring-sections) specifies that we can use `# fmt: off` and `# fmt: on` blocks or just `#fmt: skip` for one entry.

In my case, `# fmt: skip` was not enough, I had to toggle off/on.

## 5.2 Installing a python project with a pyproject.toml

- [ ] 

From [[#^spawn-howto-060957]].

2025-08-06 Wk 32 Wed - 09:57

Similar to this [guide](https://realpython.com/python-pyproject-toml/),

```sh
python3 -m pip install -e .
```

It mentions

> If all you need is to migrate from a basic [`requirements.txt` file](https://realpython.com/what-is-pip/#using-requirements-files), then you’ll be able to replace that with `pyproject.toml`.

I think that could be good for our python app here.

2025-08-06 Wk 32 Wed - 10:31

Actually in this [answer](https://stackoverflow.com/a/74625055) they're saying requirement.txt should still be present...

## 5.3 Git: How to diff between two tags

- [x] 

From [[#^spawn-howto-061103]].

2025-08-06 Wk 32 Wed - 11:03

From this [stackoverflow answer](https://stackoverflow.com/a/3211829) referencing [git-diff](https://git-scm.com/docs/git-diff),

We can just do

```sh
git diff tag1 tag2
```

Or even show commits between tags!

```sh
git log tag1..tag2
```

Or show the files changed between the two tags

```sh
git diff tag1 tag2 --stat
```

Or diffing on a specific file

```sh
git diff tag1 tag2 -- some/file/name
```


## 5.4 Git: Find out when a line in a file was changed after a commit

- [x] 

From [[#^spawn-howto-061125]].

2025-08-06 Wk 32 Wed - 11:25

From this [stackoverflow answer](https://stackoverflow.com/a/50473741/6944447),

```sh
git log -L '/the line from your file/,+1:path/to/your/file.txt'
git log -L 15,+1:'path/to/your/file.txt'
```

This [stackoverflow answer](https://stackoverflow.com/a/16635324/6944447) also covers when a file itself was changed in what commit to narrow search or deal with deleted files

```sh
git log --all -- [file path]
```

[this post](https://sentry.io/answers/view-the-change-history-of-a-single-file-in-git/) shows how track all changes across a file,

```sh
git log --follow --patch -- name-of-file
```

## 5.5 Add python project from github to requirements.txt

- [x] 

From [[#^spawn-howto-061436]].

2025-08-06 Wk 32 Wed - 14:36

From this [stackoverflow answer](https://stackoverflow.com/a/35998253/6944447),

The cleanest way is using a tag,

```sh
some-package @ git+https://github.com/owner/repo@0.1
```

2025-08-06 Wk 32 Wed - 15:39

This [stackoverflow answer](https://stackoverflow.com/a/59718369/6944447) does this but with `poetry` and `pyproject.toml`,

```
poetry add git+<protocol>://git@<repository>/<owner>/<project>.git#<git_ref>
poetry add git+ssh://git@gitlab.blabla.co/nubela/project-a.git
```

In my case,

```sh
poetry add git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4
```

You can install the package locally with 

```sh
python3 -m pip install -e .
```

or

```sh
poetry  install
```

Need to make sure versions are aligned or this happens:

```
poetry add git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4

# out
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.

Updating dependencies
Resolving dependencies... (0.0s)

The current project's supported Python range (>=3.8) is not compatible with some of the required packages Python requirement:
  - dbml-sqlite requires Python ^3.7, so it will not be installable for Python >=4.0

Because dbmint-py depends on dbml-sqlite (0.3.4) @ git+https://github.com/LanHikari22/DBML_SQLite@v0.3.4 which requires Python ^3.7, version solving failed.

  * Check your dependencies Python requirement: The Python requirement can be specified via the `python` or `markers` properties

    For dbml-sqlite, a possible solution would be to set the `python` property to ">=3.8,<4.0"

    https://python-poetry.org/docs/dependency-specification/#python-restricted-dependencies,
    https://python-poetry.org/docs/dependency-specification/#using-environment-markers
```

## 5.6 Parse a path string in python

- [x] 

From [[#^spawn-howto-30bf4d]] in [[#3.2 Implement dbmt-py]]

2025-08-15 Wk 33 Fri - 14:46

We can use the [pathvalidate](https://pypi.org/project/pathvalidate/) library which applies logic on correct paths.

From their example,

```python
import sys
from pathvalidate import ValidationError, validate_filename

try:
    validate_filename("fi:l*e/p\"a?t>h|.t<xt")
except ValidationError as e:
    print(f"{e}\n", file=sys.stderr)

```


## 5.7 Python parse enum from string with identical name to identifier

- [x] 

From [[#^spawn-howto-32f6a4]] in [[#3.2 Implement dbmt-py]]

2025-08-15 Wk 33 Fri - 16:29

[enum documentation](https://docs.python.org/3/library/enum.html)

2025-08-15 Wk 33 Fri - 16:39

You could do something like

```python
'NOTIFY' in list(TriggerType.__dict__.keys())

# out
True
```

But seems hacky...

But we can use this

```python
list(TriggerType)[0].name

# out
'NOTIFY'
```


```python
import checkpipe as pipe

enum_names = (
	TriggerType
		| pipe.OfIter[TriggerType].map(lambda e: e.name)
		| pipe.OfIter[TriggerType].filter(lambda name: name == 'NOTIFY')
		| pipe.OfIter[TriggerType].to_list()
)

enum_names

# out
['NOTIFY']
```

I noticed from this enum tutorial section on [iteration](https://docs.python.org/3/howto/enum.html#iteration) that we could convert it to a list like this.

## 5.8 Use TypeVar variables to create generic functions in python

- [x] 

From [[#^spawn-howto-862a5c]] in [[#3.2 Implement dbmt-py]]

2025-08-15 Wk 33 Fri - 17:00

From [Generics python docs](https://docs.python.org/3/library/typing.html#generics),

```python
from collections.abc import Sequence

def first[T](l: Sequence[T]) -> T:  # Function is generic over the TypeVar "T"
    return l[0]
```

```python
from collections.abc import Sequence
from typing import TypeVar

U = TypeVar('U')                  # Declare type variable "U"

def second(l: Sequence[U]) -> U:  # Function is generic over the TypeVar "U"
    return l[1]
```

I didn't know there were these two forms of syntax! I always used the latter. Let's use the former.

2025-08-15 Wk 33 Fri - 17:49

Hmm, but how do we call it and preserve the types? 

There's this syntax we used with checkpipe and is also used in the docs [here](https://docs.python.org/3/reference/compound_stmts.html#type-parameter-lists).

Similar to this,

```python
from typing import Optional

class ParseEnumLowerName[T]:
    @staticmethod
    def do(lower_name: str) -> Optional[T]:
        for e in T:
            if e.name.lower() == lower_name:
                return e
        return None
```

![[Pasted image 20250815182424.png]]

# 6 Investigations

## 6.1 Invst what happened to table.refs for pydbml

- [x] 

From [[#^spawn-invst-061055]] in [[#3.4 Open a PR for DBML_SQLite and bump version of pydbml]]

2025-08-06 Wk 32 Wed - 10:55

In commit `8fdc78b`,

With the following changes:

```diff
# in pyproject.toml
-pydbml = "^0.3.4"
+pydbml = "^1.2.0"
 
# in requirements.txt
-pydbml==0.3.4
+pydbml==1.2.0
```

We get this on testing,

```sh
python3 -m pip install -e .
poetry run pytest


# out (relevant)
tests/test_dbml_sqlite.py .F......F.F 
tests/test_terminal.py F 

>       for j, ref in enumerate(table.refs):
                                ^^^^^^^^^^
E       AttributeError: 'Table' object has no attribute 'refs'

dbml_sqlite/core.py:164: AttributeError
```

This worked in v0.3.4. So what happened to it?

Let's clone the project.

```
mkdir -p ~/src/cloned/gh/Vanderhoof/
cd ~/src/cloned/gh/Vanderhoof/
git clone git@github.com:Vanderhoof/PyDBML.git
cd PyDBML
```

2025-08-06 Wk 32 Wed - 11:02

[[#5.3 Git How to diff between two tags|HowTo]]. ^spawn-howto-061103

2025-08-06 Wk 32 Wed - 11:23

This is the [line](https://github.com/Vanderhoof/PyDBML/blob/8ffe299d2b371b6f8d286c6691e1f59c648969cb/pydbml/classes.py#L462) where refs was defined for pydbml in 0.3.4. 

So when was this line changed? [[#5.4 Git Find out when a line in a file was changed after a commit|HowTo]]. ^spawn-howto-061125

By tag 1.0.0, that file itself doesn't exist anymore.

Line was added in commit `0505a61aa6eb8e357900b030f12f8b414d96520f (tag: 0.1)`. 

2025-08-06 Wk 32 Wed - 11:44

```diff
git show 503fbe4f25b1fd8fb0eda9cf4575b81cee63dc58

# out (relevant)
--- /dev/null
+++ b/pydbml/classes/table.py

--- a/pydbml/classes.py
+++ /dev/null
```

We can see that again this file had been refactored

```sh
git log --all -- pydbml/classes/table.py

# out (relevant)
commit 90c63c83190e5e104dfbb11caa932b935fd84068
Author: Daniel Minukhin <ddddsa@gmail.com>
Date:   Thu Jul 18 20:50:56 2024 +0200

    Rewrite SQL and DBML rendering (#39)
```

```diff
git show 90c63c83190e5e104dfbb11caa932b935fd84068

# out (relevant)
--- a/pydbml/classes/table.py
+++ b/pydbml/_classes/table.py
```

Now let's track all the changes over that file 

```diff
git log --follow --patch pydbml/_classes/table.py

# out (relevant)
commit 76c45399276408f1f41f4c492814ee8e057a6f6b
Author: Vanderhoof <ddddsa@gmail.com>
Date:   Thu May 19 08:45:35 2022 +0200

    multiple schemas for tables and enums

diff --git a/pydbml/classes/table.py b/pydbml/classes/table.py
index c5847f6..f79b72a 100644
--- a/pydbml/classes/table.py
+++ b/pydbml/classes/table.py
@@ -30,21 +30,25 @@ class Table(SQLOjbect):
 
     def __init__(self,
                  name: str,
+                 schema: str = 'public',
                  alias: Optional[str] = None,
                  note: Optional[Union['Note', str]] = None,
                  header_color: Optional[str] = None,
-                 # refs: Optional[List[TableReference]] = None,
                  comment: Optional[str] = None):
         self.database: Optional[Database] = None
         self.name = name
+        self.schema = schema
         self.columns: List[Column] = []
         self.indexes: List[Index] = []
         self.alias = alias if alias else None
         self.note = Note(note)
         self.header_color = header_color
-        # self.refs = refs or []
         self.comment = comment
```

Maybe the [get_refs](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/table.py#L115) function can be used?

In [`v0.3.4 classes.py`](https://github.com/Vanderhoof/PyDBML/blob/8ffe299d2b371b6f8d286c6691e1f59c648969cb/pydbml/classes.py#L179) it says

```python
class TableReference(SQLOjbect):
    '''
    Class, representing a foreign key constraint.
    This object should be assigned to the `refs` attribute of a Table object.
    Its `sql` property contains the inline definition of the FOREIGN KEY clause.
    '''
```

In [`master _classes/reference.py`](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/reference.py#L15), 

```python
class Reference(SQLObject, DBMLObject):
    '''
    Class, representing a foreign key constraint.
    It is a separate object, which is not connected to Table or Column objects
    and its `sql` property contains the ALTER TABLE clause.
    '''
```

So it's not the same...

2025-08-06 Wk 32 Wed - 12:13

```sh
rg 'TableReference'

# out
docs/upgrading.md
27:Previously the `Table` object had a `refs` attribute which holded a list of `TableReference` objects. `TableReference` represented a table relation and duplicated the `Reference` object of `PyDBMLParseResults` container.
29:**In 1.0.0 the `TableReference` class is removed, and there's no `Table.refs` attribute.**
```

This upgrading file actually helps with upgrading projects to v1.!

They talk about refs [here](https://github.com/Vanderhoof/PyDBML/blob/master/docs/upgrading.md#getting-tables-from-parse-results-by-name).

They explain that we *can* use the `get_refs` function. The reference is defined on the table that is on the left of the relation. I believe this should also be what corresponds to `col1`.  for that reference.

2025-08-06 Wk 32 Wed - 12:27

For reference, this is my `ex000.dbml` generated sql,

```sql
CREATE TABLE Users (
  id INTEGER PRIMARY KEY,
  username TEXT,
  role TEXT,
  created_at TEXT
);
CREATE TABLE Posts (
  id INTEGER PRIMARY KEY,
  title TEXT,
  body TEXT,
  status TEXT,
  created_at TEXT
);
CREATE TABLE User_Owns_Posts (
  id INTEGER PRIMARY KEY,
  from_user_id INTEGER,
  to_post_id INTEGER,
  FOREIGN KEY(from_user_id) REFERENCES Users(id),
  FOREIGN KEY(to_post_id) REFERENCES Posts(id)
);
```

So between `User_Owns_Posts` and `Posts`, 

`col1` should be `to_post_id` and `col2` should be `id`.

and for some reason we need to do `col1[0]` to get the column because it's a single element wrapped in a list.

## 6.2 Looking at how PyDBML tests their project

- [x] 

From [[#^spawn-invst-2e2ab9]] in [[#3.7 Create a python3 library project template]]

2025-08-14 Wk 33 Thu - 18:24

Looking at how [PyDBML](https://github.com/Vanderhoof/PyDBML) do their testing.

They have many test files per feature or module, and test folders for big modules.

They have a [test_data](https://github.com/Vanderhoof/PyDBML/tree/master/test/test_data) folder with many dbml files.

They use unittest a lot, making many Test Cases. There are no instructions on how to run these tests.

They have a [test](https://github.com/Vanderhoof/PyDBML/blob/master/test/test_docs.py) for the official DBML examples.

2025-08-14 Wk 33 Thu - 18:32

This [blog](https://www.geeksforgeeks.org/python/typical-directory-structure-for-running-tests-using-unittest-in-python/) explains you can use something like

```sh
python3 -m unittest
```

Let's test this on PyDBML (we already have it cloned),

```sh
# in ~/src/cloned/gh/Vanderhoof/PyDBML
python3 -m unittest                                                                                                                                             

# out
.................................................................................................................................................................................................................................................................................................................
----------------------------------------------------------------------
Ran 305 tests in 1.196s

OK
```

Yup! We can do something like this!

2025-08-14 Wk 33 Thu - 18:41

They also mock a lot of classes with

```py
from unittest.mock import Mock
```

2025-08-14 Wk 33 Thu - 18:47

You can also notice for instance with [test_integration.py](https://github.com/Vanderhoof/PyDBML/blob/master/test/test_integration.py) that `pydbml` is an installed library  already being imported. So they maintain a separation between tests and the library. 

In order to get the `test_data/` files (example: [test_editing.py](https://github.com/Vanderhoof/PyDBML/blob/master/test/test_editing.py)), they use

```python
import os
from pathlib import Path

TEST_DATA_PATH = Path(os.path.abspath(__file__)).parent / 'test_data'
```

We can follow a similar convention. It seems good to be able to have a dedicated place for test data, especially if we are testing language, we can have many example `dbmt` files.

## 6.3 Trying to see dbml data representation for when embedding dbmt as comments

- [x] 

From [[#^spawn-invst-d0d14e]] in [[#3.2 Implement dbmt-py]]

2025-08-15 Wk 33 Fri - 10:54

For example, `// dbmt %signal get_date_now override=[insert]`. 

Doing some transformations to get python `__dict__` viewable as pretty JSON with `jq`.

```sh
python3 -m unittest tests.test_lib.DbmtPartialParsing.test_scratch \
	2>/dev/null \
	| sed -n 1p \
	| sed "s/{'/{\"/g" \
	| sed "s/, '/, \"/g" \
	| sed "s/':/\":/g" \
	| sed "s/</\"</g" \
	| sed "s/>/>\"/g" \
	| sed "s/False/\"False\"/g" \
	| sed "s/True/\"True\"/g" \
	| sed "s/None/\"None\"/g" \
	| jq

# out

# for `print(dbml_data.__dict__)`
{
  "sql_renderer": "<class 'pydbml.renderer.sql.default.renderer.DefaultSQLRenderer'>",
  "dbml_renderer": "<class 'pydbml.renderer.dbml.default.renderer.DefaultDBMLRenderer'>",
  "tables": [
    "<Table 'public' 'Users'>",
    "<Table 'public' 'Posts'>",
    "<Table 'public' 'User_Owns_Posts'>"
  ],
  "table_dict": {
    "public.Users": "<Table 'public' 'Users'>",
    "public.Posts": "<Table 'public' 'Posts'>",
    "public.User_Owns_Posts": "<Table 'public' 'User_Owns_Posts'>"
  },
  "refs": [
    "<Reference '-', ['from_user_id'], ['id']>",
    "<Reference '-', ['to_post_id'], ['id']>"
  ],
  "enums": [],
  "table_groups": [],
  "sticky_notes": [],
  "project": "None",
  "allow_properties": "False"
}
```

2025-08-15 Wk 33 Fri - 11:08

This pipeline method isn't handling quotes easily, and those dictionaries  don't support serialization  with `print(json.dumps(dbml_data.tables[0]))`, so let's just skip prettiftying for now.

Reference table:

```dbml
Table Users {
  id integer [primary key]

  // dbmt %signal is_valid_username allow=[insert, update]
  // dbmt %signal log_new_users notify=[insert]
  // dbmt %signal log_update_username notify=[update]
  username varchar

  role varchar

  // dbmt %signal get_date_now override=[insert]
  created_at varchar
}
```

Using

```python
ex_path = TEST_DATA_PATH / 'ex000_embedding_dbmt_no_types.dbml'
dbml_data = pydbml.PyDBML.parse_file(ex_path)
```

```sh
python3 -m unittest tests.test_lib.DbmtPartialParsing.test_scratch \
	2>/dev/null \
	| sed -n 1p

# out

# for `print(dbml_data.tables[0].__dict__)`
{'database': <Database>, 'name': 'Users', 'schema': 'public', 'columns': [<Column 'id', 'integer'>, <Column 'username', 'varchar'>, <Column 'role', 'varchar'>, <Column 'created_at', 'varchar'>], 'indexes': [], 'alias': None, '_note': Note(''), 'header_color': None, 'comment': None, 'abstract': False, 'properties': {}}

# for `print(dbml_data.tables[0].columns[0].__dict__)`
{'name': 'id', 'type': 'integer', 'unique': False, 'not_null': False, 'pk': True, 'autoinc': False, 'comment': None, '_note': Note(''), 'properties': {}, 'default': None, 'table': <Table 'public' 'Users'>}

# for `print(dbml_data.tables[0].columns[1].__dict__)`
{'name': 'username', 'type': 'varchar', 'unique': False, 'not_null': False, 'pk': False, 'autoinc': False, 'comment': None, '_note': Note(''), 'properties': {}, 'default': None, 'table': <Table 'public' 'Users'>}

# for `print(dbml_data.tables[0].columns[2].__dict__)`
{'name': 'role', 'type': 'varchar', 'unique': False, 'not_null': False, 'pk': False, 'autoinc': False, 'comment': None, '_note': Note(''), 'properties': {}, 'default': None, 'table': <Table 'public' 'Users'>}

# for `print(dbml_data.tables[0].columns[3].__dict__)`
{'name': 'created_at', 'type': 'varchar', 'unique': False, 'not_null': False, 'pk': False, 'autoinc': False, 'comment': None, '_note': Note(''), 'properties': {}, 'default': None, 'table': <Table 'public' 'Users'>}
```

No comments are captured. 

And yet does capture comments for tables?

```dbml
// A one-to-many relationship between users and posts
Table User_Owns_Posts {
  id integer [primary key]
  from_user_id integer [ref: - Users.id]
  to_post_id integer [ref: - Posts.id]
}
```

```sh
python3 -m unittest tests.test_lib.DbmtPartialParsing.test_scratch \
	2>/dev/null \
	| sed -n 1p

# out

# for `print(dbml_data.tables[2].__dict__)`
{'database': <Database>, 'name': 'User_Owns_Posts', 'schema': 'public', 'columns': [<Column 'id', 'integer'>, <Column 'from_user_id', 'integer'>, <Column 'to_post_id', 'integer'>], 'indexes': [], 'alias': None, '_note': Note(''), 'header_color': None, 'comment': 'A one-to-many relationship between users and posts', 'abstract': False, 'properties': {}}
```

This is unexpected behavior. But also the [dbml spec](https://dbml.dbdiagram.io/docs/#comments) doesn't really say much about how comments should be handled. It doesn't require that they are preserved in some way... but it does specify what they are.

2025-08-15 Wk 33 Fri - 11:52

We might be interested in [line @ definitions/table.py > parse_table](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/definitions/table.py#L108). They parse some `'comment_before'`.  But we can't find that being written anywhere. Let's look at what `tok` is.

[line @ definitions/enum.py > parse_enum_item](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/definitions/enum.py#L42) says that comments after settings are a priority. Could this be why?

But there's no settings in

```
  role varchar

  // dbmt %signal get_date_now override=[insert]
  created_at varchar
```

2025-08-15 Wk 33 Fri - 12:09

If you do

```
  username varchar // an inline comment
```

Then it is able to recognize it

```sh
python3 -m unittest tests.test_lib.DbmtPartialParsing.test_scratch \
        2>/dev/null \
        | sed -n 1p

# out

# for `print(dbml_data.tables[0].columns[1].__dict__)`
{'name': 'username', 'type': 'varchar', 'unique': False, 'not_null': False, 'pk': False, 'autoinc': False, 'comment': 'an inline comment', '_note': Note(''), 'properties': {}, 'default': None, 'table': <Table 'public' 'Users'>}
```

2025-08-15 Wk 33 Fri - 12:31

Filed [#63](https://github.com/Vanderhoof/PyDBML/issues/63) for the unexpected comment preservation behavior with columns.

# 7 Ideas

## 7.1 Test workflows and PyPi Version in README.md

2025-08-06 Wk 32 Wed - 14:28

The [README.md for DBML_SQLite](https://raw.githubusercontent.com/LanHikari22/DBML_SQLite/refs/heads/main/README.md) has many interactive elements to learn from. We could do this also.

## 7.2 We can contribute parsers for pyparsing

2025-08-15 Wk 33 Fri - 15:35

They seem to maintain a long list of parser examples [here](https://github.com/pyparsing/pyparsing/tree/master/examples)! Including languages, DSLs, Data formats, chemical formulas...

## 7.3 Open-collective sponsorship with pre-commit

pre-commit can be contributed to in [pre-commit](https://opencollective.com/pre-commit). We can learn from this to setup project-based funding and having multiple projects under our account there.

Also they have a game! [OpenHV](https://github.com/OpenHV/OpenHV) which is a mod for [OpenRA](https://www.openra.net/).

## 7.4 Org for a project

[pre-commit](https://github.com/pre-commit) has an org where they host the project, and the corresponding website documentation which they host. They also link the website on the org. This can be a good idea  also for dbmint. Especially since we already have multiple projects on it. 

Spawn [[002 Migrate dbmint and related to own org]] ^spawn-task-b0ef6d

# 8 Side Notes

## 8.1 Pyparsing use in pydbml

2025-08-15 Wk 33 Fri - 11:46

This library seems cool!

From [pydbml/definitions/common.py](https://github.com/Vanderhoof/PyDBML/blob/master/pydbml/definitions/common.py),

```python
import pyparsing as pp

pp.ParserElement.set_default_whitespace_chars(' \t\r')

comment = (
    pp.Suppress("//") + pp.SkipTo(pp.LineEnd())
    | pp.Suppress('/*') + ... + pp.Suppress('*/')
)
```

2025-08-15 Wk 33 Fri - 16:00

They're using `.rst` documentation (for example [here](https://github.com/pyparsing/pyparsing/blob/master/docs/HowToUsePyparsing.rst)) which points to which is this format [ReStructuredText](https://en.wikipedia.org/wiki/ReStructuredText) that is apparently used for python technical documentation.

## 8.2 Pyparsing documentation

2025-08-15 Wk 33 Fri - 14:24

Documentation for `pp.White`

```py
import pyparsing as pp

pp.White
```

says:

> Special matching class for matching whitespace. Normally, whitespace is ignored by pyparsing grammars. This class is included when some whitespace structures are significant. Define with a string containing the whitespace characters to be matched; default is `" \t\r\n"`. Also takes optional `min`, `max`, and `exact` arguments, as defined for the `Word` class.

So I guess I don't have to parse whitespace explicitly? What if I want to do exact string reconstruction for in-place writing?

## 8.3 Minimal pytest example

2025-08-15 Wk 33 Fri - 14:55

This [stackoverflow question](https://stackoverflow.com/q/47016013/6944447) gives a minimum pytest use case for reproducing an issue.

# 9 External Links

| Reference                                                                                                                                                                                                                                                            | Where                                                    |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| [Link](https://github.com/delta-domain-rnd/delta-trace/blob/main/lan/projects/2025/000%20dbmint/tasks/2025/001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md#34-open-a-pr-for--dbml_sqlite-and-bump-version-of-pydbml)                         | [#9](https://github.com/dvanderweele/DBML_SQLite/pull/9) |
| [Link](https://github.com/delta-domain-rnd/delta-trace/blob/webview/lan/projects/2025/000%20dbmint/tasks/2025/001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md#63-trying-to-see-dbml-data-representation-for-when-embedding-dbmt-as-comments) | [#63](https://github.com/Vanderhoof/PyDBML/issues/63)    |

# 10 resources