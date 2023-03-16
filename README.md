# rust_vs_swift


# Audience

This is highest-level overview possible of Rust that is created for Swift programmers and it assumes zero Rust knowledge.


# Hello world

Run in macOS terminal:
1. `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh` // downloads and installs Rust compiler and other Rust tools
2. `xcode-select --install` // you can probably omit this step - normally it's required to install linker
3. `rustc --version` // if this prints a version, you are fine
4. `mkdir hello_world`
5. `cd hello_world`
6. `vim main.rs`
```Rust
// input the following text in main.rs (press `i` key to get into insert mode in vim)
fn main() {
    // println! is not a function, it's a macro that expands during compilation
    println!("Hello, world!"); 
}
```
7. Press ESC to exit insert mode in vim and enter `:wq` to save and quit main.rs
8. `rustc main.rs`
9. `./main`

Congrats! "Hello, world!" should appear in Terminal.

Steps 4 - 9 can be shortened if we use `cargo`, which is Rust's build system and package manager:

4. `cargo new hello_world`
5. `cd hello_world`
6. `cargo run`


# Boring stuff

As per wikipedia:
- Both Swift and Rust are general-purpose languages.
- Both Swift and Rust offer imperative+procedural, object-oriented, functional and event-driven paradigms as well as generics.
- Swift offers some reflection abilities e.g. Mirror while Rust doesn't (I assume mainly due to security concerns)
- Nor Swift, neither Rust are standardized yet.


# Use cases & adoption

While Swift is a general-purpose language, it's almost exclusively used in Apple ecosystem. There is no real traction of Swift on Linux or Windows. We can't see into the future so it still can become popular on other platforms, but my personal opinion is that it will always remain a niche language that considerably small amount of programmers use (exclusively working on Apple-related apps).

Rust, in contrary, is being used everywhere nowadays: from systems-level software such as firmware / OS kernels to web, applications, game engines and server-side programming. There are few reasons for that:
- Rust is and has always been open-source and has a large and thriving open-source community and ecosystem
- It is comparable to C++ in terms of performance
- It is comparable to Swift in expressiveness and high-level coding approach (when needed)
- I.e. it gives you full control and bare metal performance when you need it; it gives you ability to write high-level code as well, which means you can apply this language everywhere. This is achieved via so called zero-cost abstractions.
- cargo - Rust's build system and package manager - is powerful and easy to use
- Rust is easily integrated with C code (due to its C ABI compliance)
- When you write Rust, in most cases there is only one standard way of doing things and you should be very explicit about your intentions, which helps spending less brainpower on picking the best approach and more on problem solving. Rust compiler is very strict and won't allow any ambiguity. There is a lot of standard tools in Rust's ecosystem (i.e. for linting, testing etc.)
- Last and most important point though, is that Rust's memory safety guarantees give big advantage when you are writing concurrent code
- Overall, developers say their Rust code uses less CPU and memory, more safe (less bugs and crashes) and more secure compared to alternative languages.

Other facts:
- Rust has been chosen as the most loved programming language for 7 years straight in Stack Overflow surveys. It jumped from 24th to 20th position on Tiobe index in 2023; while Swift is down from 12th to 15th position in the same year.
- In 2022, Rust was officially accepted as the second language (after C) into Linux kernel development. The importance of this cannot be overstated.

Non-comprehensive list of Rust adopters (listing only famous ones):
- Mozilla (they use Rust everywhere, it's their language after all)
- Google (for example, they use Rust in Android 13+ - around 1.5 million lines of code is already in Rust; 20% of new code is Rust)
- Microsoft (they are mostly concerned about memory safety/vulnerabilities and Rust is very good at solving that: https://www.theregister.com/2022/09/20/rust_microsoft_c/)
- Cloudflare (one of things they've built in Rust: https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/)
- Figma (https://www.figma.com/blog/rust-in-production-at-figma/)
- 1password (https://foundation.rust-lang.org/news/2022-03-08-member-spotlight-1password/)
- Meta, Amazon, Shopify, Atlassian, Dropbox, Coursera


# Language development
Rust Foundation is backed by Microsoft, Meta, Google, Amazon, Dropbox and others. Rust language itself is being developed by various teams in an open source fashion.


# Syntax similarity

```Rust
// Here is my implementation of MinHeap and heapsort in Rust, for example
use std::cmp;
use std::cmp::PartialOrd;
use std::fmt::Display;

pub struct MinHeap<T: Display + PartialOrd + Copy + Ord> {
    elements: Vec<T>,
}

impl<T: Display + PartialOrd + Copy + Ord> MinHeap<T> {
    pub fn new() -> Self {
        println!("Creating a new empty MinHeap");
        MinHeap {
            elements: Vec::<T>::new(),
        }
    }

    pub fn from(elements: &[T]) -> Self {
        println!("Creating a MinHeap from elements.len()={}", elements.len());
        let mut heap = MinHeap {
            elements: Vec::<T>::with_capacity(elements.len()),
        };
        for item in elements.iter() {
            heap.insert(*item);
        }
        heap
    }

    pub fn insert(&mut self, element: T) {
        self.elements.push(element);

        let mut inserted_element_index = self.elements.len() - 1;
        let mut current_parent_index = inserted_element_index / 2;

        while self.elements[inserted_element_index] < self.elements[current_parent_index] {
            self.elements.swap(inserted_element_index, current_parent_index);
            // we swapped values of inserted element with its parent,
            // so let's update the indices
            inserted_element_index = current_parent_index;
            current_parent_index = inserted_element_index / 2;
        }
    }

    pub fn extract_min(&mut self) -> Option<T> {
        if self.elements.is_empty() {
            return None;
        }

        let len = self.elements.len();
        // swap root with the last element, remove and return it (to optimize for array shifting)
        self.elements.swap(0, len - 1);
        let min_element = self.elements.remove(len - 1);
        
        // then restore heap property
        let mut inserted_element_index = 0;
        let mut left_child_index = 1;
        let mut right_child_index = 2;
        // no children = heap property restored
        // only left child = check if > than child, swap if yes and process further down
        // both left and right children exist -> check against each child and swap with the smallest one
        loop {
            let mut smallest_item_index = inserted_element_index;

            if left_child_index < self.elements.len()
                && self.elements[left_child_index] < self.elements[smallest_item_index] {
                smallest_item_index = left_child_index;
            }

            if right_child_index < self.elements.len()
                && self.elements[right_child_index] < self.elements[smallest_item_index] {
                smallest_item_index = right_child_index;
            }

            if smallest_item_index == inserted_element_index {
                // heap property restored, i.e. inserted element is <= each of its children (or there is no children)
                break;
            }

            self.elements.swap(inserted_element_index, smallest_item_index);
            inserted_element_index = smallest_item_index;
            left_child_index = 2 * inserted_element_index;
            right_child_index = 2 * inserted_element_index + 1;
        }

        return Some(min_element);
    }

    pub fn get_min(&self) -> Option<T> {
        if !self.elements.is_empty() {
            return Some(self.elements[0]);
        }
        None
    }

    pub fn len(&self) -> usize {
        self.elements.len()
    }
}

pub fn heap_sort<T: Display + PartialOrd + Ord + Copy>(arr: &[T]) -> Vec<T> {
    let mut heap = MinHeap::from(arr);
    let mut new_arr = Vec::with_capacity(arr.len());
    while let Some(min_element) = heap.extract_min() {
        new_arr.push(min_element);
    }
    new_arr
}
```

If you look at Rust code, you'd find many similarities with Swift syntax. But don't make a mistake, writing Rust is not just about knowing its syntax. Rust offers completely different memory management paradigm and this alone makes programming in Rust very different than programming in Swift and requires a ton of time to get used to and think in that new paradigm.


# Object-oriented concepts

Rust has a strong opinion about object-oriented programming where it DOESN'T have classes or inheritance at all; instead, it encourages composition over inheritance. Object-oriented aspect is achieved through trait objects and default trait implementation (similarly to how Swift offers default protocol implementation), which allows re-use of the code. Trait objects are also used for polymorphism.


# Primitive types

Rust has a similar set of signed / unsigned integers, boolean, char, floats that Swift has. Main difference is that Rust also offers 128-bit signed/unsigned types (i128 and u128).


# Zero-cost abstractions

You would not pay a penalty for the abstraction. It means whether you use the abstraction or instead go for the manual implementation you end up having the same costs (same speed, memory consumption etc.).

```
use core::mem::size_of;
struct Enabled;
let _ = size_of::<Enabled>();    // == 0
```

This ^ is just a demonstration of zero-cost abstraction principle that is done everywhere in Rust. Structures defined like this ^ are called Zero Sized Types, as they contain no actual data. Although these types act "real" at compile time - you can copy them, move them, take references to them, etc., however the compiler will completely strip them away.

In general, Rust strives to generate the same or better machine code than what you'd get if you've written it yourself. Basically, using abstractions to be more expressive do not cost you anything ABOVE what you'd have without using those abstractions.


# Performance

Because of zero-cost abstractions and rustc (compiler), in general, Rust performance is on par with C++ and even beats C in some cases.


# Ownership model & borrow checker: Rust's most loved and hated feature

## Ownership

Ownership is a set of rules that govern how a Rust program manages memory. If any of the rules are violated, the program won’t compile. None of the features of ownership will slow down your program while it’s running.

### Rules
- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

Owner goes out of scope:
```
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward
    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

String value is `moved` from one owner (s1) to another owner (s2):
```
let s1 = String::from("hello"); //string is stored on the heap
let s2 = s1;
println!("{}, world!", s1);

cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` due to previous error
```

This would compile fine tho:
```
let s1 = String::from("hello");
let s2 = s1.clone();            // clone allocates a new chunk in the heap 
                                // and deeply copies the heap data from s1
println!("s1 = {}, s2 = {}", s1, s2);
```

Stack data - copies stack data by default:
```
let x = 5;
let y = x; // compiler copies any thing that has `Copy` trait 
           // implemented incl. integers, so stack data for `x` 
           // is copied into `y` by default (no need to call x.copy() or x.clone())
println!("x = {}, y = {}", x, y);
```

Functions:
```
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here
                                    // if we tried to use s after the call to takes_ownership, 
                                    // Rust would throw a compile-time error.

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

## References and borrowing

A reference is like a pointer in that it’s an address we can follow to access the data stored at that address; that data is owned by some other variable. Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type for the life of that reference.

References allow you to refer to some value without taking ownership of it. Because a reference does not own the data, the value it points to will not be dropped when the reference is destroyed.

```
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);  // &s1 is an immutable reference to s1
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

We call the action of creating a reference borrowing. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back. You don’t own it.

So, what happens if we try to modify something we’re borrowing?
```
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}

$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error
```

This one works:
```
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

However:
```
 let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);


$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
error: could not compile `ownership` due to previous error
```

The restriction preventing multiple mutable references to the same data at the same time allows for mutation but in a very controlled fashion. It’s something that new Rustaceans struggle with because most languages let you mutate whenever you’d like. The benefit of having this restriction is that Rust can prevent data races at compile time. A data race is similar to a race condition and happens when these three behaviors occur:
- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There’s no mechanism being used to synchronize access to the data.

Data races cause undefined behavior and can be difficult to diagnose and fix when you’re trying to track them down at runtime; Rust prevents this problem by refusing to compile code with data races!

As always, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not simultaneous ones:
```
let mut s = String::from("hello");

{
    let r1 = &mut s;
} // r1 goes out of scope here, so we can make a new reference with no problems.

let r2 = &mut s;
```

Rust enforces a similar rule for combining mutable and immutable references. This code results in an error:
```
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);


$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error

```

## Dangling references

In languages with pointers, it’s easy to erroneously create a dangling pointer — a pointer that references a location in memory that may have been given to someone else — by freeing some memory while preserving a pointer to that memory. In Rust, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

Let’s try to create a dangling reference to see how Rust prevents them with a compile-time error:
```
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle returns a reference to a String
    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!


$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error
```

## References rules
- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.


# Memory safety

In safe Rust, which is what you use by default:
- It is not possible to dereference a null pointer.
- It is not possible to use a dangling pointer.
- It is not possible to forget to free memory.
- It is not possible to free already-freed memory.

There is also `unsafe` Rust that you could [opt-in to when needed](https://doc.rust-lang.org/nomicon/intro.html), but that's a different story


# Thread Safety

Rust's type system prevents data races at compile time (Send and Sync traits).
You can't read and write the same variable from multiple threads at the same time without wrapping it in a lock or other concurrency primitive. That'd introduce a data race bug and the compiler won't allow it.
You can't forget to acquire a lock before accessing the variable it protects.


# Backward compatibility

Release of Rust 1.0 in 2015 established stability without stagnation principle. The rule for Rust has been that once a feature has been released on stable, compiler team is committed to supporting that feature for ALL future releases.
If breaking change is needed it is released as an `Edition` which is opt-in. So nothing breaks unless you explicitly migrate and opt-in into a new edition.

See more here: https://doc.rust-lang.org/edition-guide/editions/index.html


# Tools

Rust provides compiler, build system and package manager (cargo) once default installation is done.
Cargo includes support for unit testing out of the box (no need for complex setup - just write unit tests in code and run via `cargo test`).

There is support for Rust in various IDEs, Visual Studio Code being one of the most favorited by community.
Additionally one can install rustfmt (linter with default rules), clippy (advisor for idiomatic code), cargodoc (for generating documentation) etc.

See more here https://www.rust-lang.org/tools


# Platform support

Rust compiler supports a lot of platforms as an output target. 
In theory, anything LLVM supports Rust can support too.
See more here https://doc.rust-lang.org/rustc/platform-support.html


# Where Swift has advantages over Rust

- If your need is to exclusively write software for Apple platform(s), Swift is the best choice
- Swift has considerably smaller learning curve than Rust. You'd start writing programs in Swift much faster than in Rust (maybe almost an order of magnitude faster)
- In case of a single-threaded app, Swift would be a better choice, as there won't be concurrency issues. Due to its memory safety guarantees, Rust compiler enforces a lot of rules on a programmer and you consistently need to fight the compiler. For multi-threaded apps this provides cure against concurrency-related problems and is needed evil. However, if your app is simple enough and uses only main thread, Rust would still enforce all the rules on you and make your life hard while Swift will be easy to use and at the same time safe (as no other threads exist).


# Where Rust has advantages over Swift

- If you need to compile and run on multiple platforms
- Where maximum performance is needed
- Where there are low limits on how much RAM usage is allowed
- Where highly concurrent code exist (basically any complex app)
- Where memory safety is critical (e.g. apps where security is critical - and nowadays it's a big amount of apps - that's why Rust becomes widely accepted and used)


# Swift -> Rust calls

Rust is C ABI compatible so essentially Swift interoperates with Rust in a way similar to interoperation with C.
In Rust you'd need to expose APIs as C APIs via `extern "C"` stuff for public Rust interfaces and mark them with `#[no_mangle]` compiler directive so that compiler doesn't change symbol names.
In Swift we'd need to use bridging header to include those Rust APIs (exposed as C APIs).


# Rust -> Swift calls

There is undocumented/unofficial `@_cdecl` compiler directive in Swift that would expose it for languages supporting C FFI including Rust. On Rust side, we'd need to wrap Swift function into `extern C` block and then would need to write a Rust-y function that wraps this `extern C` call to provide friendly interface to native Rust code.


# How to learn Rust

Start with [The Rust Programming Language](https://doc.rust-lang.org/book/) and go from there. Use exercises in the book to get familiar with the language.

Explore [Rust By Example](https://doc.rust-lang.org/stable/rust-by-example/).

Do the [Rustlings exercises to get used to reading and writing Rust](https://github.com/rust-lang/rustlings/).

Then, as always, go build something.


# Comments
Comments and questions are welcome. I'm not an expert in this field and could make mistakes.


# References

- https://en.wikipedia.org/wiki/Comparison_of_programming_languages
- https://www.rust-lang.org/
- https://doc.rust-lang.org/book/title-page.html
- https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html
- https://doc.rust-lang.org/book/ch16-00-concurrency.html
- https://blog.rust-lang.org/2014/10/30/Stability.html
- https://faq.sealedabstract.com/rust/
- https://doc.rust-lang.org/nomicon/intro.html