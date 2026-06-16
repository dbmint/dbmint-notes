---
parent: '[[001 Create Python and Rust parsing for dbmt language]]'
spawned_by: '[[001 Create Python and Rust parsing for dbmt language]]'
context_type: task
status: todo
---

Parent: [001 Create Python and Rust parsing for dbmt language](../001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md)

Spawned by: [001 Create Python and Rust parsing for dbmt language](../001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md)

Spawned in: [<a name="spawn-task-c758ce" />^spawn-task-c758ce](../001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md#spawn-task-c758ce)

# 1 Journal

This is for [PR #9](https://github.com/dvanderweele/DBML_SQLite/pull/9).

2025-08-06 Wk 32 Wed - 09:23

Associated issue is [issue #8](https://github.com/dvanderweele/DBML_SQLite/issues/8) for [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/) to bump version.

Let's fork [gh dvanderweele/DBML_SQLite](https://github.com/dvanderweele/DBML_SQLite/).

````sh
mkdir -p ~/src/cloned/gh/LanHikari22/forked/dvanderweele
cd ~/src/cloned/gh/LanHikari22/forked/dvanderweele
git clone git@github.com:LanHikari22/DBML_SQLite.git
cd DBML_SQLite
````

Let's also correct our convention of `forked/` for other projects. We should always include the users or organizations that own the project

````
mkdir -p ~/src/cloned/gh/LanHikari22/forked/apache/
mv ~/src/cloned/gh/LanHikari22/forked/arrow-rs ~/src/cloned/gh/LanHikari22/forked/apache/arrow-rs
mkdir -p ~/src/cloned/gh/LanHikari22/forked/ron-rs
mv ~/src/cloned/gh/LanHikari22/forked/ron  ~/src/cloned/gh/LanHikari22/forked/ron-rs
````

Let's create a fix branch for bumping the version.

````sh
git checkout -b fix-bump-pydbml-v1.2.0
````

````diff
# in requirements.txt
-pydbml==0.3.4
+pydbml==1.2.0
````

We should run the tests recommended in the [README.md](https://github.com/dvanderweele/DBML_SQLite/?tab=readme-ov-file#testing-and-coverage),

We need to install [poetry](https://python-poetry.org/) first

````sh
python3 -m pip install poetry
````

````sh
poetry run pytest

# out (exit=1)
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.
Creating virtualenv dbml-sqlite-Sv97MqvY-py3.13 in /home/lan/.cache/pypoetry/virtualenvs
Command not found: pytest
````

````sh
poetry run coverage run --source dbml_sqlite -m pytest

# out (exit=1)
The "poetry.dev-dependencies" section is deprecated and will be removed in a future version. Use "poetry.group.dev.dependencies" instead.
Command not found: coverage
````

Let's install pytest and install the fork

````sh
python3 -m pip install pytest
````

2025-08-06 Wk 32 Wed - 09:56

[000 Installing a python project with a pyproject.toml](../howtos/000%20Installing%20a%20python%20project%20with%20a%20pyproject.toml.md)

2025-08-06 Wk 32 Wed - 10:14

````sh
python3 -m pip install -e .
````

````sh
poetry run pytest

# out (relevant)
def test_process_column():

>       assert processColumn(c2, 'full') == '  c2 INTEGER PRIMARY KEY'

tests/test_dbml_sqlite.py:93: 

>               if column.autoinc:
E               AttributeError: 'MockColumn' object has no attribute 'autoinc'

dbml_sqlite/core.py:216: AttributeError
````

<a name="pr-4-tests-failing" />^pr-4-tests-failing

This issue occurs even on pydbml v0.3.4.

I've been modifying requirement.txt version, but not pyproject.toml... Either way will stay v0.3.4 for now to see if we can get the tests to run.

2025-08-06 Wk 32 Wed - 10:35

````sh
git blame dbml_sqlite/core.py

# out (relevant)
6e0c970e (Philipp Hörist   2023-06-14 20:32:37 +0200 216)             if column.autoinc:
6e0c970e (Philipp Hörist   2023-06-14 20:32:37 +0200 217)                 segments.append(' AUTOINCREMENT')
````

Seems to be due to a latest PR change?

````
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
````

2025-08-06 Wk 32 Wed - 10:41

This is PR [\#4](https://github.com/dvanderweele/DBML_SQLite/pull/4). The author [states](https://github.com/dvanderweele/DBML_SQLite/pull/4#issuecomment-2499196571) there that they have abandoned the project and have not tested this PR and may turn it read-only.

Let's go to the commit before the PR and see if tests run ok

````sh
git checkout -b 8fdc78bb23f2ffe12b9be583d1792d25032aae5f
python3 -m pip install -e .
poetry run pytest

# out (relevant)
tests/test_dbml_sqlite.py ...........                                                                                                                                                  [ 91%]
tests/test_terminal.py .                                                                                                                                                               [100%]

12 passed in 0.46s
````

They work fine.

What about after bumping versions?

````diff
# in pyproject.toml
-pydbml = "^0.3.4"
+pydbml = "^1.2.0"
 
# in requirements.txt
-pydbml==0.3.4
+pydbml==1.2.0
````

````sh
python3 -m pip install -e .
poetry run pytest


# out (relevant)
tests/test_dbml_sqlite.py .F......F.F 
tests/test_terminal.py F 

>       for j, ref in enumerate(table.refs):
                                ^^^^^^^^^^
E       AttributeError: 'Table' object has no attribute 'refs'

dbml_sqlite/core.py:164: AttributeError
````

2025-08-06 Wk 32 Wed - 10:55

[001 Create Python and Rust parsing for dbmt language > 6.1 Invst what happened to table.refs for pydbml](../001%20Create%20Python%20and%20Rust%20parsing%20for%20dbmt%20language.md#61-invst-what-happened-to-tablerefs-for-pydbml)

2025-08-06 Wk 32 Wed - 12:01

There is a [get_refs](https://github.com/Vanderhoof/PyDBML/blob/5c6a9173580d7273a6799cdd725eabd3a2e76911/pydbml/_classes/table.py#L115) function that can still be used. Let's change refs to that.

2025-08-06 Wk 32 Wed - 12:32

````diff
# in dbml_sqlite/core.py

@@ -186,7 +186,8 @@ def processRef(ref, join=True):
-    segments.append(f'{ref.col.name}) REFERENCES {ref.ref_table.name}({ref.ref_col.name})')
+    segments.append(f'{ref.col1[0].name}) REFERENCES {ref.col2[0].table.name}({ref.col2[0].name})')
````

2025-08-06 Wk 32 Wed - 12:57

Luckily the PR [\#4](https://github.com/dvanderweele/DBML_SQLite/pull/4) issue wasn't so bad. Just had to add an extra field to the mock class to get the tests to run.

Let's use the same convention [here](https://github.com/dvanderweele/DBML_SQLite/pull/4#issue-1757442972) where we write in the PR `Fixes #N` for the issue ID N.

2025-08-06 Wk 32 Wed - 13:08

````sh
remote: Create a pull request for 'fix-bump-pydbml-v1.2.0' on GitHub by visiting:
remote:      https://github.com/LanHikari22/DBML_SQLite/pull/new/fix-bump-pydbml-v1.2.0
````

Let's do it!

See [\#9](https://github.com/dvanderweele/DBML_SQLite/pull/9).

2025-08-06 Wk 32 Wed - 13:37

This might be merged in. There are open PRs on this project that fix various issues. Let's try to merge them ourselves into the fork master branch.

## 1.1 Merge currently open PRs into fork main for DBML_SQLite

2025-08-06 Wk 32 Wed - 13:57

This [stackoverflow answer](https://stackoverflow.com/questions/1425892/how-do-you-merge-two-git-repositories#10548919) could help us merge those PRs into our fork.

Let's get the PRs,

We want [\#6](https://github.com/dvanderweele/DBML_SQLite/pull/6) and [\#7](https://github.com/dvanderweele/DBML_SQLite/pull/7).

Let's merge our PR in our main,

````sh
# in /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite
# in main branch
git merge fix-bump-pydbml-v1.2.0
````

Let's start with [\#6](https://github.com/dvanderweele/DBML_SQLite/pull/6),

````sh
mkdir -p ~/src/cloned/gh/dvanderweele/forks/KoStyle/
cd ~/src/cloned/gh/dvanderweele/forks/KoStyle/
git clone git@github.com:KoStyle/DBML_SQLite.git
cd /home/lan/src/cloned/gh/LanHikari22/forked/dvanderweele/DBML_SQLite

# in main branch
git remote add sibling ~/src/cloned/gh/dvanderweele/forks/KoStyle/DBML_SQLite
git fetch sibling
git merge sibling/main
git remote remove sibling
````

We do pass all tests via

````sh
python3 -m pip install -e .
poetry run pytest
````

Repeating for [\#7](https://github.com/dvanderweele/DBML_SQLite/pull/7),

````sh
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
````

All tests pass.

2025-08-06 Wk 32 Wed - 14:19

I guess I should've just mentioned #6 and #7. Github is able to reference them on my fork.

![Pasted image 20250806141949.png](../../../../../attachments/Pasted%20image%2020250806141949.png)

Name is a bit weird, but it's alright. It'd take force pushing to change this, and it's synced with three sources.
