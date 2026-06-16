---
parent: '[[005 Create proc macro to abstract over diesel and extension manual creation]]'
spawned_by: '[[005 Create proc macro to abstract over diesel and extension manual creation]]'
context_type: task
status: todo
---

Parent: [005 Create proc macro to abstract over diesel and extension manual creation](../005%20Create%20proc%20macro%20to%20abstract%20over%20diesel%20and%20extension%20manual%20creation.md)

Spawned by: [005 Create proc macro to abstract over diesel and extension manual creation](../005%20Create%20proc%20macro%20to%20abstract%20over%20diesel%20and%20extension%20manual%20creation.md)

Spawned in: [<a name="spawn-task-7ee363" />^spawn-task-7ee363](../005%20Create%20proc%20macro%20to%20abstract%20over%20diesel%20and%20extension%20manual%20creation.md#spawn-task-7ee363)

# 1 Journal

2025-11-18 Wk 47 Tue - 11:38 +03:00

[rust-by-example derive](https://doc.rust-lang.org/rust-by-example/trait/derive.html)

2025-11-18 Wk 47 Tue - 11:57 +03:00

Following [gh imbolc/rust-derive-macro-guide](https://github.com/imbolc/rust-derive-macro-guide),

2025-11-18 Wk 47 Tue - 12:01 +03:00

Let's create a temporary project:

````
mkdir -p ~/tmp/del
cd ~/tmp/del
rm -rf *
````

````
cargo install cargo-edit
cargo install cargo-expand
````

We need to create two crates, a tester and a derive macro one.

````sh
# in /home/lan/tmp/del
cargo new --lib mytrait-derive-tester
cargo new --lib mytrait-derive
````

````sh
# in /home/lan/tmp/del/mytrait-derive-tester
cargo add mytrait-derive --path ~/tmp/del/mytrait-derive
````

````sh
# in /home/lan/tmp/del
cat >> mytrait-derive/Cargo.toml << EOF
[lib]
proc-macro = true
EOF
````

````sh
# in /home/lan/tmp/del/mytrait-derive
cargo add proc-macro2@1.0 quote@1.0
cargo add syn@1.0 --features full
````

````rust
// in /home/lan/tmp/del/mytrait-derive/src/lib.rs

use proc_macro::{self, TokenStream};
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyTrait)]
pub fn derive(input: TokenStream) -> TokenStream {
    let DeriveInput { ident, .. } = parse_macro_input!(input);
    let output = quote! {
        impl MyTrait for #ident {}
    };
    output.into()
}
````

2025-11-18 Wk 47 Tue - 13:19 +03:00

Now we can testing by putting

````rust
// in /home/lan/tmp/del/mytrait-derive-tester/src/lib.rs
use mytrait_derive::MyTrait;

trait MyTrait {
    fn answer() -> i32 {
        42
    }
}

#[derive(MyTrait)]
struct Foo;
````

2025-11-18 Wk 47 Tue - 13:31 +03:00

````sh
# in /home/lan/tmp/del/mytrait-derive
cargo build

# out
   Compiling mytrait-derive v0.1.0 (/home/lan/tmp/del/mytrait-derive)
error: `proc-macro` crate types currently cannot export any items other than functions tagged with `#[proc_macro]`, `#[proc_macro_derive]`, or `#[proc_macro_attribute]`
  --> src/lib.rs:14:1
   |
14 | pub fn add(left: u64, right: u64) -> u64 {
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error: could not compile `mytrait-derive` (lib) due to 1 previous error
````

We need to remove the code that was already in `/home/lan/tmp/del/mytrait-derive/src/lib.rs` too, so that this is the only content:

````rust
// in /home/lan/tmp/del/mytrait-derive/src/lib.rs
use proc_macro::{self, TokenStream};
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyTrait)]
pub fn derive(input: TokenStream) -> TokenStream {
    let DeriveInput { ident, .. } = parse_macro_input!(input);
    let output = quote! {
        impl MyTrait for #ident {}
    };
    output.into()
}
````

````sh
# in /home/lan/tmp/del/mytrait-derive
cargo build

# out
   Compiling mytrait-derive v0.1.0 (/home/lan/tmp/del/mytrait-derive)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.46s
````

So back to testing in `mytrait-derive-tester`:

````sh
# in /home/lan/tmp/del/mytrait-derive-tester
cargo expand

# out
    Checking mytrait-derive-tester v0.1.0 (/home/lan/tmp/del/mytrait-derive-tester)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.14s

#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2024::*;
#[macro_use]
extern crate std;
use mytrait_derive::MyTrait;
trait MyTrait {
    fn answer() -> i32 {
        42
    }
}
struct Foo;
impl MyTrait for Foo {}
pub fn add(left: u64, right: u64) -> u64 {
    left + right
}
````

2025-11-18 Wk 47 Tue - 13:41 +03:00

For parsing of initialization values:

````sh
# in /home/lan/tmp/del/mytrait-derive
cargo add darling@0.13
````

````rust
// in /home/lan/tmp/del/mytrait-derive/src/lib.rs
use darling::FromDeriveInput;
use proc_macro::{self, TokenStream};
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[derive(FromDeriveInput, Default)]
#[darling(default, attributes(my_trait))]
struct Opts {
    answer: Option<i32>,
}

#[proc_macro_derive(MyTrait, attributes(my_trait))]
pub fn derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input);
    let opts = match Opts::from_derive_input(&input) {
        Ok(opts) => opts,
        Err(err) => return err.write_errors().into(),
    };

    let DeriveInput { ident, .. } = input;

    let answer = match opts.answer {
        Some(x) => quote! {
            fn answer() -> i32 {
                #x
            }
        },
        None => quote! {},
    };

    let output = quote! {
        impl MyTrait for #ident {
            #answer
        }
    };
    output.into()
}
````

As a diff:

````diff
+use darling::FromDeriveInput;
 use proc_macro::{self, TokenStream};
 use quote::quote;
 use syn::{parse_macro_input, DeriveInput};

-#[proc_macro_derive(MyTrait)]
+#[derive(FromDeriveInput, Default)]
+#[darling(default, attributes(my_trait))]
+struct Opts {
+    answer: Option<i32>,
+}
+
+#[proc_macro_derive(MyTrait, attributes(my_trait))]
 pub fn derive(input: TokenStream) -> TokenStream {
-    let DeriveInput { ident, .. } = parse_macro_input!(input);
+    let input = parse_macro_input!(input);
+    let opts = match Opts::from_derive_input(&input) {
+        Ok(opts) => opts,
+        Err(err) => return err.write_errors().into(),
+    };
+
+    let DeriveInput { ident, .. } = input;
+
+    let answer = match opts.answer {
+        Some(x) => quote! {
+            fn answer() -> i32 {
+                #x
+            }
+        },
+        None => quote! {},
+    };
+
     let output = quote! {
-        impl MyTrait for #ident {}
+        impl MyTrait for #ident {
+            #answer
+        }
     };
     output.into()
 }
````

2025-11-18 Wk 47 Tue - 14:12 +03:00

````rust
// in /home/lan/tmp/del/mytrait-derive-tester/src/lib.rs
use mytrait_derive::MyTrait;

trait MyTrait {
    fn answer() -> i32 {
        42
    }
}

#[derive(MyTrait)]
struct Foo;

#[derive(MyTrait)]
#[my_trait(answer = 0)]
struct Bar;

#[test]
fn default() {
    assert_eq!(Foo::answer(), 42);
}

#[test]
fn getter() {
    assert_eq!(Bar::answer(), 0);
}
````

````sh
# in /home/lan/tmp/del/mytrait-derive-tester
cargo expand

# out
    Checking mytrait-derive-tester v0.1.0 (/home/lan/tmp/del/mytrait-derive-tester)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.14s

#![feature(prelude_import)]
#[prelude_import]
use std::prelude::rust_2024::*;
#[macro_use]
extern crate std;
use mytrait_derive::MyTrait;
trait MyTrait {
    fn answer() -> i32 {
        42
    }
}
struct Foo;
impl MyTrait for Foo {}
#[my_trait(answer = 0)]
struct Bar;
impl MyTrait for Bar {
    fn answer() -> i32 {
        0i32
    }
}
````

`cargo test` all pass. Now we can have attributes at the struct level and we can create a basic derive.

One use case of interest is settings applied to a struct member.
