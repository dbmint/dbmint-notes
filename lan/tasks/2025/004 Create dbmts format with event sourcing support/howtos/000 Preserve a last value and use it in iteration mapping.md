---
parent: '[[004 Create dbmts format with event sourcing support]]'
spawned_by: '[[002 Group settings by table and enforce some conditions on the table and settings for dbmts]]'
context_type: howto
status: done
---

Parent: [004 Create dbmts format with event sourcing support](../004%20Create%20dbmts%20format%20with%20event%20sourcing%20support.md)

Spawned by: [002 Group settings by table and enforce some conditions on the table and settings for dbmts](../tasks/002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md)

Spawned in: [<a name="spawn-howto-ecf5e1" />^spawn-howto-ecf5e1](../tasks/002%20Group%20settings%20by%20table%20and%20enforce%20some%20conditions%20on%20the%20table%20and%20settings%20for%20dbmts.md#spawn-howto-ecf5e1)

# 1 Objective

in Rust with itertools, I would like to be able to "keep the last value".

I have

````
(Some(1), None)
(None, Some(1))
(None, Some(2))
(None, Some(3))
(Some(2), None)
(None, Some(1))

...
````

I would like to be able to just imprint the Some(1) and Some(2) "down" into the remaining items like so:

````
(Some(1), None)
(Some(1), Some(1))
(Some(1), Some(2))
(Some(1), Some(3))
(Some(2), None)
(Some(2), Some(1))
````

# 2 Journal

2025-10-09 Wk 41 Thu - 19:45 +03:00

(llm)

Using LLM (ChatGPT 5),

[docs.rs core scan](https://doc.rust-lang.org/stable/core/iter/trait.Iterator.html#method.scan)

(/llm)

[docs.rs core scan](https://doc.rust-lang.org/stable/core/iter/trait.Iterator.html#method.scan) should be able to help. It's like [docs.rs core fold](https://doc.rust-lang.org/stable/core/iter/trait.Iterator.html#method.fold) but we're not accumulating, just keeping some internal state.

2025-10-09 Wk 41 Thu - 20:02 +03:00

````rust
// in /home/lan/src/cloned/gh/LanHikari22/rs_repro/src/bin/expt000_howto_ecf5e1.rs

fn main() {
    let input = [
        (Some(1), None),
        (None, Some(1)),
        (None, Some(2)),
        (None, Some(3)),
        (Some(2), None),
        (None, Some(1)),
    ];

    let output = input
        .into_iter()
        .scan(None, |state, (left, right)| {
            if let Some(left) = left {
                *state = Some(left);
            }

            Some((*state, right))
        })
        .collect::<Vec<_>>();

    println!("{output:?}");
}
````

````sh
# in /home/lan/src/cloned/gh/LanHikari22/rs_repro/
cargo run --bin expt000_howto_ecf5e1

# out
[(Some(1), None), (Some(1), Some(1)), (Some(1), Some(2)), (Some(1), Some(3)), (Some(2), None), (Some(2), Some(1))]
````

2025-10-09 Wk 41 Thu - 20:04 +03:00

OK

2025-10-10 Wk 41 Fri - 02:39 +03:00

````sh
# in /home/lan/src/cloned/gh/dbmint/dbmts_rs
git commit -m "event sourcing derive implemented"

# out
Trim Trailing Whitespace.................................................Passed
Check Yaml...........................................(no files to check)Skipped
Check for added large files..............................................Passed
Check formatting.........................................................Passed
Run tests................................................................Passed
Check clippy lints.......................................................Passed
[main 36aca04] event sourcing derive implemented
 7 files changed, 1221 insertions(+), 42 deletions(-)
 create mode 100644 src/dbmts_parser.rs
 create mode 100644 src/event_sourcing.rs
 create mode 100644 src/main.rs
````
