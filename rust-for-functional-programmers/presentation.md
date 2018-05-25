name: inverse
layout: true
class: center, middle, inverse
---
# Rust for Functional Programmers

Predictable behavior meets predictable performance

---
layout: false
# Agenda

* Intro to Rust and why it matters

* Linear types, kind of

* Functional Rust

* Odds and ends

---
template:inverse
# Intro to Rust and why it matters

---
layout: false
## History

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

--

* 2018: and again

---
## What is it?

* A systems programming language

* Memory-, type-, and concurrency-safe

* Statically typed, with parametric polymorphism

* Expression oriented

* Strictly evaluated

* Impure

---
## What is systems programming?

* Programming for resource-constrained and/or performance-sensitive environments

* Examples
    * Medical devices

    * Automotive systems

    * Real-time audio/video processing

    * Game engines

    * Virtual/Augmented reality

---
## Why not C/C++?

* Riddled with undefined behavior
    * Uninitialized values

    * Unchecked reads and writes

    * Double frees

* Notoriously insecure

* Concurrency is a nightmare

* Template error messages from hell

---
## An example of undefined behavior

```c++
// What do you think this will do?
int main(int, char**) {
  unsigned long a[1];
  a[3] = 0x7ffff7b23e61UL;
  (void) a;
  return 0;
}
```

(adapted from Blandy and Orendorff's Programming Rust)

---
## Why not (insert favorite high-level language)?

* Sometimes you need to control the "how", not just the "what"

* Portability can be a challenge

* Garbage collection has downsides

---
## What's wrong with garbage collection?

* Fundamental time/space tradeoff

* Unpredictable pause times

* Unreliable for non-memory resources

---
template:inverse
# Linear types, kind of

---
layout: false
## Substructural type systems in brief

* Linear type: value is used exactly once

* Affine type: value is used at most once

* Relevant type: value is used at least once

---
## The ownership model: linear types, kind of

* Every live value must have exactly one owner

* A value may be borrowed as long as it is owned, but no longer

* A mutable borrow may not overlap other borrows

* Immutable borrows may overlap other immutable borrows

---
## Substructural Rust

* Linear, usually: values are disposed of exactly once by default

* Affine, always: values will never be disposed of more than once, but might not be disposed of at all (e.g. mem::forget, abort)

* Relevant, sometimes: `#[must_use]` tells compiler to warn if value unused

---
## Borrowing example (broken)

```rust
let mut v = vec![1, 2, 3];

for i in &v {           // immutable borrow created
    println!("{}", i);
    v.push(34);         // ERROR: requires a mutable borrow
}                       // immutable borrow released
```

---
## Borrowing example (fixed)

```rust
let mut v = vec![1, 2, 3];

for i in &v.clone() {   // immutable borrow of clone created
    println!("{}", i);
    v.push(34);         // okay; we're not modifying the clone
}                       // immutable borrow of clone released
```

---
## Lifetimes: parametric poly-scope-ism

* Types containing references must be parameterized by the lifetime(s) of those references

* Ditto for functions that return references (although they can sometimes be inferred)

```rust
trait Bar<'a> {
    fn bar(&self) -> &'a str;
}

fn foo<'a, T: Bar<'a>>(arg: T) -> &'a str {
    arg.bar()
}
```

---
## Safe concurrency: no more data races

* Rust supports multiple safe concurrency paradigms
    * Message passing

    * Fork/join

    * Shared memory
    
* Free from data races thanks to mutexes that own the data they protect

* But not free from race conditions!

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
template:inverse
# Functional Rust

---
layout: false
## Functional Rust

* Immutable by default

* Expression oriented

* Pattern matching

* Hindley-Milner type inference

* Traits AKA typeclasses

* Associated types (a limited form of functional dependencies)

* No higher kinded types yet (but it's being worked on)

---
## Monadic types and traits

* Option: Maybe

* Result: Either

* Iterator, Future, Stream, etc.

---
## Principled mutability

* Global mutable state is prohibited

* Immutable access is always referentially transparent

* Mutable access means exclusive access

---
## Im: Immutable data structures for Rust

* Immutable lists, bitmapped vector tries, hash array mapped tries, and B-trees

* Cache friendly

* Shared structure via reference counting

* Relies on `Arc::make_mut` for safe destructive updates

---
template:inverse
# Odds and ends

---
layout: false
## Other cool crates

* Rayon: deterministic parallel iteration

* Crossbeam-STM: software transactional memory

* Futures: lightweight, asynchronous control flow

* Tokio: asynchronous I/O

* Stdweb: typesafe bindings to Web (browser) APIs

---
## WebAssembly

* Run and debug Rust code in your browser

* wasm-pack: publish NPM packages written in Rust

* Nebulet and Cervus: sandboxed apps in kernel space

---
## Unsafe Rust

* Code that can't be proven safe by the compiler must be wrapped in an `unsafe` block

* Indicates that the programmer is responsible for safety, not the compiler

* Useful for implementing high-performance data structures and FFI bindings

---
## Caveats

* High learning curve

* Tiny community compared to e.g. Clojure or Scala

* Many popular crates are changing rapidly, with unstable APIs

* Somewhat verbose

---
## Resources

* https://play.rust-lang.org

* https://webassembly.studio

* https://reddit.com/r/rust

* https://stackoverflow.com/questions/tagged/rust

* https://www.meetup.com/Rust-Boulder-Denver

---
template:inverse
# Finito
