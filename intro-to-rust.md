name: inverse
layout: true
class: center, middle, inverse
---
# Intro to Rust

Safe, efficient, and practical

---
layout: false
# Agenda

* History

* Features

* Rust vs. C++

* Rust at Mersive

---
## History

--

* 2006: project started by Graydon Hoare

--

* 2009: sponsored by Mozilla

--

* 2012: first pre-alpha release: 0.1

--

* 2015: first stable release: 1.0

--

* 2016: "most loved" language on StackOverflow

--

* 2017: ditto

---
## Adoption

* Mozilla: Servo and Quantum

* Dropbox: Magic Pocket

* Google: parts of ChromeOS and Fucshia

* Facebook: Mononoke

* Microsoft: search feature in Visual Studio Code

---
## Features

* fast

* memory-safe

* thread-safe

* practical

---
.left-column[
## Features

* .black[**fast**]

* memory-safe

* thread-safe

* practical
]
.right-column[
## Fast
{{content}}
]

--

* minimal runtime
{{content}}

--

* compiles to machine code
{{content}}

--

* no garbage collector
{{content}}

--

* static dispatch by default
{{content}}

--

* no implicit copies by default

---
.left-column[
## Features

* .black[**fast**]

* memory-safe

* thread-safe

* practical
]
.right-column[
## Why not GC?

* unpredictable pauses

* 2x or more memory bloat

* pointer heavy -> bad locality of reference

* handles non-memory resources poorly
]

---
.left-column[
## Features

* fast

* .black[**memory-safe**]

* thread-safe

* practical
]
.right-column[
## Memory-safe
{{content}}
]

--

* single-owner move semantics by default
{{content}}

--

* non-nullable types
{{content}}

--

* safe borrowing via references
{{content}}

--

* reference counting

---
.left-column[
## Features

* fast

* .black[**memory-safe**]

* thread-safe

* practical
]
.right-column[
## Implications

* no segfaults

* no uninitialized values

* no dangling pointers

* no memory corruption
]

---
.left-column[
## Features

* fast

* .black[**memory-safe**]

* thread-safe

* practical
]
.right-column[
## Ownership rules

* every live value must have exactly one owner

* a value may only be borrowed for as long as it is owned

* a mutable borrow may not overlap other borrows

* immutable borrows may overlap other immutable borrows
]

---
## Borrow checker example (broken)

```rust
let mut v = vec![1, 2, 3];

for i in &v {           // immutable borrow created
    println!("{}", i);
    v.push(34);         // ERROR: requires a mutable borrow
}                       // immutable borrow released
```

---
## Borrow checker example (fixed)

```rust
let mut v = vec![1, 2, 3];

for i in &v.clone() {   // immutable borrow of clone created
    println!("{}", i);
    v.push(34);         // okay; we're not modifying the clone
}                       // immutable borrow of clone released
```

---
.left-column[
## Features

* fast

* memory-safe

* .black[**thread-safe**]

* practical
]
.right-column[
## Thread-safe
{{content}}
]

--

* values are immutable by default
{{content}}

--

* shared state disallowed by default
{{content}}

--

* Mutex + owned values = guaranteed safety

---
# Thread example (broken)

```rust
use std::collections::HashMap;
use std::thread::spawn;

fn main() {
    let mut map = HashMap::new();
    let thread = spawn(|| {
        map.insert("foo", "bar");
    });

    map.insert("yo", "yup");

    thread.join().unwrap();

    assert_eq!(Some(&"yup"), map.get("yo"));
    assert_eq!(Some(&"bar"), map.get("foo"));
}

```

---
# Thread example (fixed)

```rust
use std::collections::HashMap;
use std::thread::spawn;
use std::sync::{Arc, Mutex};

fn main() {
    let map = Arc::new(Mutex::new(HashMap::new()));
    let clone = map.clone();
    let thread = spawn(move || {
        clone.lock().unwrap().insert("foo", "bar");
    });

    map.lock().unwrap().insert("yo", "yup");

    thread.join().unwrap();

    let m = map.lock().unwrap();

    assert_eq!(Some(&"yup"), m.get("yo"));
    assert_eq!(Some(&"bar"), m.get("foo"));

    println!("success!");
}
```

---
.left-column[
## Features

* fast

* memory-safe

* thread-safe

* .black[**practical**]
]
.right-column[
## Practical
{{content}}
]

--

* portable
{{content}}

--

* modern package manager
{{content}}

--

* helpful error messages
{{content}}

--

* hygenic macros
{{content}}

--

* friendly community

---
## Why not C++?

--

* header files suck

--

* preprocessor macros are evil

--

* undefined behavior bites us again and again
    * segfaults
    * data races
    * heisenbugs
    * security holes

--

* opaque error messages

--

* no dependency management

---
## Rust drawbacks

--

* still new and changing quickly

--

* relatively difficult to learn

--

* error handling can be awkward

--

* primitive C/C++ interoperability

--

* slow compiler

---
## Kepler Local Monitoring Service

* watches Solstice JSON log and forwards events to the cloud

* separate process -> immune to Solstice crashes

* no runtime dependencies

* compile-time dependencies managed by cargo

---
## Conclusion

* good choice for new projects

* doesn't play nice with C++ (yet)

* still rough, but promising future
