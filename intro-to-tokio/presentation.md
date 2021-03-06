name: inverse
layout: true
class: center, middle, inverse
---
# Intro to Tokio

High performance I/O for Rust

---
layout: false
# Agenda

* A brief history of I/O

* Futures, streams, sinks, and combinators

* Mio, Tokio, Hyper, etc.

* The future of futures: async/await

---
template:inverse
# Caution: Unstable APIs Ahead

---
layout: false
## A Brief History of I/O

--

* Before we had threads, we had non-blocking I/O, but it was hard to use

--

* Then we had threads, but they had their own problems

    * 10,000 threads * 8MB stacks = 80GB

    * context switches ain't free

    * neither is cache coherency

--

* So we went back to non-blocking

--

    * (plus high-level, low-overhead abstractions to keep things sane)

---
![Thread context switch costs](http://ithare.com/wp-content/uploads/part101_infographics_v08.png)

---
## Concurrency Models

--

* Blocking I/O with pre-emptive threads

    * Pros: easy to program, good multicore parallelism (when done properly)

    * Cons: non-deterministic, high memory use, context switch overhead

--

* Non-blocking I/O with stackful coroutines

    * Pros: deterministic, easy to program, low context switch overhead

    * Cons: single core, high memory use, incompatible with Rust std

--

* Non-blocking I/O with stackless coroutines

    * Pros: deterministic, minimal context switch overhead, low memory use

    * Cons: single core, harder to program (but getting easier)

--

* Hybrid: Non-blocking I/O with a pre-emptive thread pool (M:N)

    * Pros: multicore, maximum performance for I/O- and CPU-heavy workloads

    * Cons: non-deterministic, harder to program (but good abstractions make it manageable)

---
## Concurrency Models in Rust

* Blocking I/O with pre-emptive threads

    * std::net, Hyper (<0.11), Iron

* Non-blocking I/O with stackful coroutines (not recommended)

    * mioco, coio-rs, Corona

* Non-blocking I/O with stackless coroutines; optional M:N threading

    * Tokio, Hyper (>=0.11), Gotham

---
template:inverse
# What is a Future?

--

## An asynchronously pollable placeholder for a result that may not yet be available

---
layout: false
## Futures/Promises in Various Languages

* Java: Future/CompletableFuture

* JavaScript: Promise

* C#: Task/TaskCompletionSource

* C++: std::future

* Rust: futures::Future

---
template:inverse
# What is a Stream?

--

## An asynchronously pollable iterator over a series of values

---
template:inverse
# What is a Sink?

--

## An asynchronously flushable destination for a series of values

---
template:inverse
# What do I do with all these things?

--

## Combinate them!

---
layout: false

## Examples of Combinators

--

* `map`: transform the result of a future (or stream) if/when it's available

--

* `filter_map`: as above, but optionally filter out some of the values

--

* `and_then`: handle the value in a future (or stream) asynchronously, returning a new Future

--

* `or_else`: if the first future (or stream) fails, try the second one instead

--

* `join`: combine one or more futures into a single future that's fulfilled when all the inputs are fulfilled

--

* `select`: as above, but the result is fulfilled when *any* of the inputs are fulfilled

--

* `fold`: reduce the values in a stream to a single value

--

* `unfold`: generate a stream of values from an asynchronous state machine

---
# Code Example

```rust
/// Returns a future which will download the contents of the specified
/// URLs in parallel and send them to the specified sinks.
pub fn download_all<C: Connect>(
    client: Client<C>,
    urls_and_sinks: Vec<(Uri, impl Sink<SinkItem = Chunk, SinkError = Error>)>,
    timeout: Duration,
) -> impl Future<Item = (), Error = Error> {
    let timer = Timer::default();

    join_all(urls_and_sinks.into_iter().map(move |(url, sink)| {
        client
            .get(url)
            .from_err()
            .and_then(|response| response.body().from_err().forward(sink))
            .select(
                timer
                    .sleep(timeout)
                    .from_err()
                    .and_then(|_| err(format_err!("request timeout"))),
            )
            .map_err(|(e, _)| e)
    })).map(drop)
}
```

---
## Mio: Metal I/O

Low-level cross-platform, non-blocking I/O abstraction for TCP and UDP

* Linux: epoll

* MacOS/iOS: kqueue

* Windows: I/O completion ports

* etc.

---
## Tokio

* Futures-based abstraction on top of Mio

* Old API: tokio-core, tokio-io (<0.1.5), tokio-proto

* Newer API: tokio, tokio-io (>=0.1.5)

* Newest API: tokio (>=0.2.0)

---
## Handy Tokio-based and Futures-based Libraries

* Hyper: HTTP clients and servers

* Warp: High-level server framework based on Hyper

* Rust-WebSocket: WebSocket clients and servers

* Rdkafka: Kafka connectivity

---
## The Future of Futures

Current, combinator style:

```rust
pub fn fetch_rust_lang<C: Connect>(client: Client<C>) -> impl Future<Item = String, Error = Error> {
    result("https://www.rust-lang.org".parse())
        .from_err()
        .and_then(move |uri| {
            client.get(uri).from_err().and_then(|response| {
                if response.status().is_success() {
                    Box::new(
                        response.body().concat2().from_err().and_then(
                            |body| {
                                result(from_utf8(&body).map(str::to_string))
                                    .from_err()
                            },
                        ),
                    ) as Box<dyn Future<Item = String, Error = Error>>
                } else {
                    Box::new(err(format_err!(
                        "request failed with status: {}",
                        response.status()
                    )))
                }
            })
        }),
}
```

---
## The Future of Futures

Async/await style:

```rust
pub async fn fetch_rust_lang<C: Connect>(client: Client<C>) -> Result<String, Error> {
    let response = client.get("https://www.rust-lang.org".parse()?).await?;
    if response.status().is_success() {
        let body = response.body().concat2().await?;
        Ok(from_utf8(body)?.to_string())
    } else {
        Err(format_err!("request failed with status: {}", response.status()))
    }
}
```

---
## Resources

* Intro and rationale: http://aturon.github.io/blog/2016/08/11/futures/

* The road to Futures 1.0: http://aturon.github.io/2018/02/27/futures-0-2-RC/

* Progress towards async/await: https://areweasyncyet.rs/

* In-progress book: _Asynchronous Programming in Rust_: https://rust-lang.github.io/async-book/

* Tokio docs and how-tos: https://tokio.rs/

* Third-party benchmarks: https://www.techempower.com/benchmarks/#section=data-r15&hw=ph&test=plaintext
