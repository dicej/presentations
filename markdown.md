name: inverse
layout: true
class: center, middle, inverse
---
# Tail Calls and Continuations on the JVM

Mmm... Spaghetti Stacks

---
layout: false
# Agenda

* Tail Call Optimization: Reduce, Reuse, Recycle

* First Class Continuations: Time Travel Without Paradoxes

* Fun With Trampolines

* A Renegade JVM

---
template:inverse
## What's a tail call?

---
layout: false
## When the last thing a function does before returning is call a function, that's a tail call.

```java
int foo() {
  notATailCall();

  int n = alsoNotATailCall();

  if (n == 42) {
    return thisIsATailCall();
  } else {
    n = 3;
  }

  if (bar()) {
    return n + notATailCall();
  }

  synchronized (this) {
    return notATailCall();
  }
}
```

---
template:inverse
## What's tail call optimization?

---
layout: false
## Tail call optimization means supporting arbitrary tail recursion within a fixed size call stack.

### Before

![Call stack without Tail Call Optimization](http://www.billhails.net/Book/images/tail-no-tco.png)

### After

![Call stack with Tail Call Optimization](http://www.billhails.net/Book/images/tail-with-tco.png)

---
template:inverse
## Why should I care?

---
layout: false
.left-column[
## Why TCO?
]
.right-column[
* Without TCO, the only way to do arbitrarily large computations is to iterate

* Iteration implies mutable state, and We Don't Like That&trade;

* You can hide the iteration inside a higher order function like
`fold`, but sometimes you need the full power of explicit
recursion

]
---
template:inverse
## And now, a story about a sandwich...

---
background-image: url(https://ashleyfurniture.scene7.com/is/image/AshleyFurniture/B636-78-76-99)

---
background-image: url(https://services-homedepot-com.s3.amazonaws.com/services/kitchen-desing-ideas.jpg)

---
background-image: url(http://kathysbakery.com/wp-content/uploads/2016/10/bakery.jpg)

---
background-image: url(https://d2zd0z9rcn78un.cloudfront.net/sites/default/files/styles/homepage_height_620px/public/media/franchises/food_drink_images/tk.com_bbyv_homepage2_2c.jpg?itok=1wpYN5BQ)

---
background-image: url(https://s-media-cache-ak0.pinimg.com/originals/d2/53/fe/d253fe2eddae0824d6d386f15dccd157.jpg)

---
background-image: url(https://www.bbcgoodfood.com/sites/default/files/glossary/banana-crop.jpg)

---
## Continuations: time travel without paradoxes

* A continuation is snapshot of execution state

* Can be resumed multiple times

* Paradox-free

  * Each execution of a continuation is independent of the others

  * If you go back and kill your grandfather to prevent your own birth, you will have created a new, forked timeline without affecting the one you came from

---
## Continuation uses

* Replace state machines

* End callback hell

* Error handling

* Dynamic reoptimization

* Serverless webapps

---
## Delimited Continuations

* Capture a delimited subset of the call stack

* Can be called and composed like functions

```java
void sandwichStory() {
  reset(() -> {
    eat(shift(makeSandwichAsync));
    takeNap();
  });
  doOtherStuff();
}
```

---
## Fun facts about continuations

* First-class continuations are equivalent in power to monads

* The continuation monad is the mother of all monads -- you can derive *all* other monads from it

* Can be used to implement generators, coroutines, green threads, etc.

* Can be serialized for execution elsewhere and elsewhen

---
template: inverse
## What does this look like on the JVM?

---
layout: false
## The JVM won't help us with TCO

This can't be represented in Java bytecode:

![Call stack with Tail Call Optimization](http://www.billhails.net/Book/images/tail-with-tco.png)

  (There is a `goto` bytecode, but it can only jump within a function)

---
## Tail calls in Clojure: self calls

Use `recur`, which is converted to iteration at compile time

.left-half[Wrong:

```clojure
(defn foo [n]
  (if (< n 1000000000)
    (foo (+ n 1))
    n))
```
]

.right-half[Right:

```clojure
(defn foo [n]
  (if (< n 1000000000)
    (recur (+ n 1))
    n))
```
]
---
## Tail calls in Clojure: mutual calls

Use `trampoline`, which takes a function that returns either another
function or a value.  It keeps calling any functions returned until
one of them returns a value.

.left-half[Wrong:

```clojure
(defn foo [n]
  (if (< n 1000000000)
    (bar (+ n 1))
    n))

(defn bar [n]
  (if (< n 1000000000)
    (foo (+ n 2))
    n))

(foo 2)
```
]

.right-half[Right:

```clojure
(defn foo [n]
  (if (< n 1000000000)
    (fn [] (bar (+ n 1)))
    n))

(defn bar [n]
  (if (< n 1000000000)
    (fn [] (foo (+ n 2)))
    n))

(trampoline foo 2)
```
]

---
## The JVM won't help us with continuations, either

* You can't get a reference to the call stack

* Even you could, it would be mutable, so you'd also need some way to atomically copy it to and from the heap

---
## Delimited continuations in Scala

Selectively transforms code into continuation passing style at compile time

.left-half[Original:

```scala
def foo() = {
  1 + shift(k => k(k(k(7))))
}
def bar() = {
  foo() * 2
}
def baz() = {
  reset(bar())
}
baz()  // returns 70
```
]

.right-half[Transformed:

```scala
reset(new Shift(k => k(k(k(7))))
        .map(x => (x + 1) * 2))
```
]

---
## Other ways to emulate first class continuations

* Use your own stack instead of (or in addition to) the call stack

* Bookmarking and exceptions (e.g. Jetty continuations)

* ContT monad transformer

---
template: inverse
## What if they were built into the JVM?

---
layout: false
## OpenJDK's Multi-Language VM (MLVM) project

* Introduced `invokedynamic` and lambda support

* Also aimed to add continuations, stack introspection, and TCO, but those projects seem to have stalled

* HotSpot (Oracle's JVM) is big and complicated, so adding those features is big job

---
## Ain't nobody got time for that

* I wrote a JVM for various reasons: https://github.com/ReadyTalk/avian

* 10x smaller and 10x stupider than HotSpot

* Supports native TCO and first class continuations

* Ran both Scala and Clojure last time I checked, but that was a while ago

---
## Avian TCO

* Avian may be built with or without TCO

* Without TCO: caller pushes and pops arguments in own frame

* With TCO: caller pushes arguments; callee pops them

  * When executing a tail call, callee pops own arguments, pushes new callee's arguments, and jumps to the new callee

  * Original caller doesn't know or care where control is returned from

---
.left-half[
## Avian continuations

* A linked stack (AKA spaghetti stack) is more efficient for capturing and calling continuations frequently

* Otherwise, an array-based stack is better

* Best of both worlds: a hybrid stack
]
.right-half[
![Spaghetti Stack](https://upload.wikimedia.org/wikipedia/commons/e/e5/Spaghettistack.svg)
]


---
We use the native call stack exclusively until/unless a continuation is captured:

```
native call stack: [frame A] [frame B] [frame C]
```

--
To capture a continuation (call it `[cont X]`):

1. Copy the call stack into a heap-allocated linked list -- the head of this list is the continuation

    ```
    native call stack: [frame A] [frame B] [frame C]

                 heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
    ```
--
2. Remove all but the most recent frame from the native stack, and replace them with a pointer to the heap-allocated list.

    ```
    native call stack: [] [frame C']
                        \______________
                                       \
                                       \/
                 heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
    ```

    Note that the copy of `[frame C']` on the stack may have been modified, which we indicate by relabeling it `[frame C']`.

---
Now imagine our program has run a little longer, and the stack looks like this:

```
native call stack: [] [frame C'] [frame D] [frame E]
                    \______________
                                   \
                                   \/
             heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
```
--
Let's grab another continuation (call it `[cont Y]`):

```
native call stack: [] [frame E']
                    \__________________
                                       \
                                       \/
             heap: .- [frame C'] <- [frame D] <- [frame E] <- [cont Y]
                   |___________________
                                       \
                                       \/
                       [frame A] <- [frame B] <- [frame C] <- [cont X]
```
---
Now let's call `[cont X]`:

1. Pop any frames that don't belong to `[cont X]`

    ```
    native call stack: []
                        \______________
                                       \
                                       \/
                 heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
                                       /\
                   .___________________/
                   |
                   '- [frame C'] <- [frame D] <- [frame E] <- [cont Y]
    ```
--
2. Push any frames that do belong to `[cont X]`

    ```
    native call stack: [] [frame C'']
                        \______________
                                       \
                                       \/
                 heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
                                       /\
                   .___________________/
                   |
                   '- [frame C'] <- [frame D] <- [frame E] <- [cont Y]
    ```
---
And finally, let's call `[cont Y]`:

1. Pop any frames that don't belong to `[cont Y]`

    ```
    native call stack: []
                        \______________
                                       \
                                       \/
                 heap: [frame A] <- [frame B] <- [frame C] <- [cont X]
                                       /\
                   .___________________/
                   |
                   '- [frame C'] <- [frame D] <- [frame E] <- [cont Y]
    ```
--
2. Push any frames that do belong to `[cont Y]`

    ```
    native call stack: [] [frame E']
                        \__________________
                                           \
                                           \/
                 heap: .- [frame C'] <- [frame D] <- [frame E] <- [cont Y]
                       |___________________
                                           \
                                           \/
                           [frame A] <- [frame B] <- [frame C] <- [cont X]
    ```

---
template: inverse
## Questions?

---
## References

[The Mother of all Monads](http://blog.sigfpe.com/2008/12/mother-of-all-monads.html)

[Scala Delimited Continuations via CPS-Transform](https://infoscience.epfl.ch/record/149136/files/icfp113-rompf.pdf)

[Jetty Continuations](http://www.eclipse.org/jetty/documentation/9.4.x/continuations-using.html)

[Avian Continuations JavaDoc](https://readytalk.github.io/avian-web/javadoc-1.2.0/avian/Continuations.html)
