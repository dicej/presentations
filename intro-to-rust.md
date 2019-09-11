name: inverse
layout: true
class: center, middle, inverse
---
# Rust Learning Group Kickoff

---
layout: false
# Agenda

* Intro to Rust

   * History
   * Features
   * Drawbacks
   * Rust at Mersive

* Pick a time to meet

* Pick a book to read

---
## History

* 2006: project started by Graydon Hoare

* 2009: sponsored by Mozilla

* 2012: first pre-alpha release: 0.1

* 2015: first stable release: 1.0

* 2016 - 2019: "most loved" language on StackOverflow

---
## Adoption

* Mozilla: Servo and Quantum

* Dropbox: Magic Pocket

* Google: parts of ChromeOS and Fucshia

* Facebook: Mononoke, Libra

* Microsoft: parts of Azure, parts of Visual Studio Code, MSRC

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

* minimal runtime
{{content}}

* compiles to machine code
{{content}}

* no garbage collector
{{content}}

* static dispatch by default
{{content}}

* no implicit copies (instead: move, slice, or borrow)
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

* single-owner move semantics by default
{{content}}

* non-nullable types
{{content}}

* safe borrowing via references
{{content}}

* reference counting
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

* values are immutable by default
{{content}}

* shared state disallowed by default
{{content}}

* Mutex + owned values = guaranteed safety
]

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

* portable
{{content}}

* modern package manager
{{content}}

* helpful error messages
{{content}}

* friendly community
]

---
## Other Features

--

* functional
    * expression oriented
    * algebraic data types
    * pattern matching
    * lambdas
    * higher-order functions

--

* generic

* traits (AKA interfaces, AKA typeclasses)

* hygenic macros and derivations (i.e. no name collisions)

* open types (i.e. add new methods to existing types)

---
## Algebraic Data Type Examples

```rust
// product type
struct Person {
    first_name: String,
    last_name: String,
    birth_date: DateTime
}

// sum type
enum MessageType {
    Hello,
    Ping,
    Goodbye
}

// sum of product types
enum Message {
    Hello(Uuid),
    Ping(Uuid, i32),
    Goodbye {
      id: Uuid,
      message: String
      timestamp: DateTime
    }
}
```

---
## Pattern Match Example

```rust
// sum of product types
enum Message {
    Hello(Uuid),
    Ping(Uuid, i32),
    Goodbye {
      id: Uuid,
      message: String
      timestamp: DateTime
    }
}

match message {
    Hello(id) => {
        eprintln!("got a Hello with id = {}", id);
    }

    Ping(id, sequence) => {
        eprintln!("got a Ping with id = {}, sequence = {}", id, sequence);
    }

    Goodbye { id, message, timestamp } => {
        eprintln!("got a Goodbye with id = {}, message = {}, and timestamp = {}",
                  id, message, timestamp);
    }
}
```

---
## Trait Derivation Example

```rust
#[derive(Serialize, Deserialize, PartialEq, Debug)]
#[serde(rename_all = "camelCase")]
struct TaskState {
    message_type: MessageType,
    task_id: Uuid,
    task_type: TaskType,
    status: Status,
    download_progress: Option<u64>,
    download_total: Option<u64>,
    error_message: Option<String>,
}
```

---
## Rust Drawbacks

* still new and changing quickly

* steep learning curve

* primitive C++ interoperability

* slow compiler (though not bad compared to C++)

* people like me who won't shut up about how great it is

---
## Rust at Mersive

* Kepler

    * Local Monitoring Service (LMS)
    * SQL Marshaling
    * Email Service
    * Dashboard Onboarding Service

* Solstice Discovery Service (SDS)

* WebRTC Server

* OpenControl API v2 Server

---
## Resources

* https://doc.rust-lang.org/book/

* https://play.rust-lang.org

* https://reddit.com/r/rust

* https://stackoverflow.com/questions/tagged/rust

---
## When shall we meet?

* day?

* morning, lunchtime, or end of day?

---
## Which book shall we read?

* The Rust Programming Language (https://doc.rust-lang.org/book/)

* Programming Rust (http://shop.oreilly.com/product/0636920040385.do)

* Something else?
