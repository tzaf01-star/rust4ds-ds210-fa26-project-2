# Project 2: Build Your Own Vec

**Author:** Timur Zafesov



### Getting started

You will build Rust's `Vec` yourself, twice. First a slow version that is safe and simple, then a fast one that manages its own memory with raw pointers. Then you will time the versions against each other.

Each vector has a playground you can run and edit freely. Run these from this top folder:

```bash
cargo run --bin slow_playground   # try out SlowVec and FixedSizeArray
cargo run --bin slow_memory       # watch what gets dropped, and when
cargo run --bin fast_playground   # try out FastVec, MALLOC, and raw pointers
cargo run --bin fast_memory       # watch what gets dropped and freed, and when
```

All four crash with `not yet implemented` until you write the functions they call.

### The folders

```
slow_vec/    Part 1. Your SlowVec goes in src/lib.rs
fast_vec/    Part 2. Your FastVec goes in src/lib.rs, plus src/growth.rs and src/timing.rs
             src/benchmark.rs runs the benchmark and draws the plots
fixed/       FixedSizeArray, which SlowVec is built on. Read it if you are curious
malloc/      the memory allocator FastVec uses. It records every allocation and free
tracker/     wraps values so you can see which ones have been dropped
```

**Change only what the handout asks for**: the functions marked `todo!`, the `Drop` for `FastVec`, your choices in `benchmark.rs`, and the writeup at the bottom of this file. Leave everything else alone.

### About the warnings

A fresh clone builds with a lot of warnings, all of them some version of "you declared this and never used it." That is expected, since you haven't written the code yet.

### Running your tests

There is one test file per checkpoint. Run them from this top folder:

```bash
cargo test --test cp1     # SlowVec
cargo test --test cp2     # FastVec's get and push, and growth
cargo test --test cp3     # remove, clear, Drop, and timing
cargo test                # all of them
```

Add part of a test's name to run just that one: `cargo test --test cp2 push_memory`.

**If a run stops partway with `SIGSEGV`, `SIGBUS`, `SIGABRT`, or some other signal**, one of your `unsafe` blocks has corrupted memory, and it took the whole test run down with it. The last test named before the crash is the place to start. Try running it on its own to diagnose the problem.

### The benchmark

```bash
cargo run --release --bin benchmark
```

Remember, the `--release` matters: without it the timings are slow and misleading. This writes `benchmark_slow.png` and `benchmark_growth.png` into the top folder, which you'll need for the writeup. Commit both: they have to be on `main` when you submit.

## Your writeup

Answer the questions from the project handout here, under the headings below. Leave the headings where they are, and leave everything above this line alone. Answers should be about 2-3 sentences per question unless the handout says otherwise.

### 1. SlowVec against FastVec
SlowVec execution time grows increasingly rapid, forming an upward-curivung graph, while FasVec remains almosr flat at the same scale. THat happend because SlowVec reallocates and moves all exsiting elements with every push, giving a total cost of n(n+1)/2 or n^2 while FastVec doubling is O(n)
### 2. Why push is fast on average
For 16 pushes, FastVec with doubling performs 16 writes for the new elements and 15 additional moves during reallocations (1 + 2 + 4 + 8), totaling 31 operations. Although some pushes are expensive because they require moving existing elements into a larger allocation, these operations happen less frequently as capacity increases. 
### 3. Your growth experiments
I tested five growth settings across two experiments: Double, Multiply(1.5), Multiply(1.25), Add(10,000), and Add(100,000). In the multiplicative experiment, Double was the fastest at approximately 1.66 ms, followed by Multiply(1.5) at 2.00 ms and Multiply(1.25) at 2.78 ms; in the additive experiment, Double again performed best at approximately 1.65 ms, compared with 2.8 ms for Add(100,000) and 19.8 ms for Add(10,000).
### 4. Why `Add` eventually loses
With Add(k), capacity increases by a fixed number of slots, meaning that reallocations continue at regular intervals even as the vector becomes much larger. This leads to expensive copying then the number of items grows leading to a O(n^2)
### 5. Two kinds of leak
Tracker checks whether individual elements are properly dropped, while MALLOC checks whether the vector's allocated memory blocks are freed. A destructor that frees the vector's buffer but fails to drop its elements could pass MALLOC's checks while failing Tracker's checks. Both are necessary because releasing an allocation does not automatically clean up resources owned by the values stored inside it.
### 6. The bug in `clear`
The original FastVec clear() function freed the vector's allocated buffer without dropping the individual elements, potentially causing memory leaks when those elements owned additional resources. I fixed this by iterating through the live elements, using ptr::read() to move each value out and drop() to clean it up before releasing the buffer and resetting the vector's fields.
### 7. Something that failed at first, and how you adapted
Mostly everything was failing first, but after some time thinking about what to fix and how + google questions and AI help I adapted to the problem. I always tried to understand the root cause of the issues arising, which was helpful and brought me a better understanding of ownership and memroy allocation.
### AI citation
Project 2.1 I used AI for guidence and verification of my code as well as assistant on understanding of how and why does the code work and does not. 
Project 2.2 I used ChatGPT to help implement Growth::next_capacity(), FastVec::get(), and FastVec::push(). ChatGPT provided example code, explained Rust concepts including raw pointers, memory allocation, and ownership, and helped debug compilation errors. I used these explanations to understand the implementation and troubleshoot my code.
Project 2.3 I used ChatGPT to guide me through the code writing for checkpoint 3. AI helped me understand the differences between some strategies i was trying beforehand.