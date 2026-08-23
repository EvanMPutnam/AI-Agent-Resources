# 1. Rust

Standards for Rust projects. They extend [coding-rules.md](../common/coding-rules.md) and
override it where the two disagree. Everything here applies to production code; § 8 marks
what relaxes in tests.

This file is not global. A Rust project adopts it whole or not at all.

## 1.1 Core Principles

- Prefer correctness, explicitness, and maintainability over cleverness.
- Make the compiler do the checking. A rule the type system enforces beats a rule in this document.
- Keep modules focused and composable.
- Design for deterministic behavior first. Optimize after profiling.
- Treat warnings as failures in CI.

## 1.2 Errors and Panics

- No `unwrap()` or `expect()` in production code.
- Return `Result<T, E>` for fallible operations and propagate with `?`.
- Return `Option<T>` when absence is expected and unexceptional. Convert to `Result` with context where the absence is a failure.
- Define domain error enums with `thiserror`. Use `anyhow` in binaries and tests, never in a library's public API.
- Implement `source()` so the error chain survives. Never build errors by formatting strings into a `String` variant.
- Add context at each boundary the error crosses, not at every `?`.
- `panic!`, `assert!`, and slice indexing state invariants the programmer controls. They are not error handling, and they must not appear on a path a user or a file can reach.
- Use `debug_assert!` for invariant checks too expensive for release.
- Prefer `.get(i)` over `v[i]` anywhere the index is computed rather than proven.
- Set `#[must_use]` on types and functions where dropping the value is a bug.
- A library must document every panic it can still reach, under a `# Panics` heading.

## 1.3 Types and API Design

- Make invalid states unrepresentable. An enum with data beats a struct of optional fields plus a rule about which combinations are legal.
- Use newtypes for values that must not be interchangeable — addresses, cycle counts, indices into different arrays. `struct Addr(u16)` costs nothing at runtime.
- Default to private. Widen to `pub(crate)`, then `pub`, only when a caller exists.
- No boolean arguments. `f(true, false)` says nothing at the call site; take an enum or a config struct.
- Take borrowed inputs (`&str`, `&[T]`, `&T`) and return owned. Take `impl Into<String>` only where callers routinely hold both forms.
- Use `Cow<'_, str>` when a function usually borrows and occasionally allocates.
- Convert with `From`/`TryFrom`. Reserve `as` for § 1.5.
- Derive `Debug` on every public type. Derive `Clone`, `Copy`, `PartialEq`, and `Default` where the semantics are real, not because the derive list looks incomplete.
- Mark public enums and structs `#[non_exhaustive]` when adding a variant or field later should not be a breaking change.
- Seal a trait that exists for internal dispatch, so downstream crates cannot implement it.
- Implement `Deref` only for smart pointers. Using it for inheritance hides where methods come from.

## 1.4 Ownership and Borrowing

- Minimize cloning in hot paths. Clone freely in setup, configuration, and error paths where the cost is unmeasurable.
- Prefer stack allocation and fixed-size arrays for state with a known bound.
- Avoid shared mutable state. Prefer passing ownership, then borrows, then channels, and only then a lock.
- Use interior mutability (`Cell`, `RefCell`, `Mutex`) with a comment saying why the ordinary form does not work. `RefCell` moves a compile error to a runtime panic.
- Use iterators idiomatically, but do not chain adapters past the point where a reader can say what the pipeline produces.
- Do not depend on subtle drop order. Fields drop in declaration order; if that ordering carries meaning, make it explicit with a scope or an explicit `drop`.

## 1.5 Numerics and Casts

- `as` between integer types truncates silently. Use `From` when the conversion is lossless and `TryFrom` when it can fail.
- Where truncation is the intent, say so: `value as u8` earns a comment, or use `u8::try_from(v).expect(..)` in tests and a masked `wrapping` form in production.
- Integer overflow panics in debug and wraps in release. Never rely on either default. Write `wrapping_add`, `checked_add`, or `saturating_add` and pick deliberately.
- Turn on `overflow-checks = true` in the release profile unless a benchmark shows it costs too much.
- Name magic bit patterns. `const HALF_CARRY: u8 = 0x20;` beats `0x20` at four call sites.
- Convert bytes with explicit endianness — `u16::from_le_bytes`, `to_be_bytes`. Never transmute a byte slice into an integer.
- Avoid `f32`/`f64` for values that must compare or reproduce exactly.

## 1.6 Traits and Generics

- Reach for a concrete type first, a generic when a second caller needs a second type, and `dyn Trait` when the set of types is open or monomorphization bloats the binary.
- Keep trait methods few and orthogonal. A trait with a dozen required methods is a struct wearing a costume.
- Provide default method bodies where an implementor would otherwise copy the same code.
- Do not add a trait to enable mocking. Test against the real type, or extract the boundary the trait was standing in for.
- Prefer `impl Trait` in return position over boxing when the caller does not need to name the type.

## 1.7 Modules and Project Structure

- Organize by domain, not by layer. `cpu`, `ppu`, `bus`, `cartridge` — not `traits`, `structs`, `helpers`.
- Keep IO and parsing at the edges. The core logic should be callable from a test with no filesystem, clock, or network.
- Split a module when it stops having one describable job, not when it passes a line count.
- Use `mod.rs`-free layout (`cpu.rs` beside `cpu/`) consistently across the project.
- Put shared lint configuration in `[workspace.lints]` in `Cargo.toml`, not in crate-root attributes duplicated per crate.

## 1.8 Macros

- Write a function first. A macro is for what a function cannot express: variadic call sites, code generation across many types, or capturing an expression unevaluated.
- Prefer `macro_rules!` over a proc macro until the pattern needs to parse Rust syntax.
- A macro that generates public API must generate documentation with it.
- Do not use a macro to hide control flow that returns from the caller, except for a well-named, documented early-return idiom.

## 1.9 Style and Documentation

- Follow `rustfmt` defaults. Never hand-format against the tool.
- Use descriptive names. Domain-standard abbreviations (`pc`, `sp`, `ppu`, `mmu`) are names, not abbreviations.
- Write explicit types where inference makes a reader scroll to find out what they have.
- Document every public item with `///`, including a `# Errors` section for anything returning `Result` and `# Panics` for anything that can panic.
- Write examples as doctests so the compiler checks them.
- Set `#![warn(missing_docs)]` on a library crate with external users.
- Comment intent. The code already states mechanics.
- Where an implementation follows a specification, cite the section — `// LR35902 §3.2: HALT bug skips the PC increment`.

# 2. Concurrency

- Prefer a design with no shared state. Move ownership between threads, or pass messages over a channel.
- Where a lock is needed, keep the critical section short and the lock private to the module that owns the data.
- Never hold a `std::sync::Mutex` guard across `.await`. Use `tokio::sync::Mutex` when a lock must span a suspension point, and question the design first.
- Do not block in async code. Move filesystem work, CPU-bound work, and blocking C calls to `spawn_blocking` or a dedicated thread.
- Every `select!` branch must be cancellation-safe, since a losing branch is dropped mid-future. Read each future's documentation for that guarantee rather than assuming it.
- Give every spawned task a shutdown path. A task that outlives its owner is a leak.
- Do not implement `Send` or `Sync` by hand. That is an `unsafe` claim about the whole type; see § 3.
- Test concurrent code with a deterministic executor or a loom model where the interleaving matters. A passing stress test proves nothing about a race.

# 3. Unsafe Code

- Set `#![forbid(unsafe_code)]` on any crate that does not need it, so the exemption is visible in review.
- `unsafe` requires a measured need, not a suspected one. Benchmark the safe version first.
- Every `unsafe` block carries a `// SAFETY:` comment naming the invariants that make it sound and why they hold here.
- Enable `unsafe_op_in_unsafe_fn` so an `unsafe fn` body does not silently inherit permission.
- Keep the unsafe surface small: one module, a safe API around it, invariants documented on the boundary type.
- Run `cargo miri test` over any crate containing `unsafe`. Undefined behavior does not fail a normal test run.
- Review unsafe changes for aliasing, alignment, initialization, and lifetimes specifically, not as ordinary code.

# 4. Testing

- Unit tests for state transitions and single behaviors. Integration tests for subsystem interaction. End-to-end tests against real inputs for compatibility.
- Tests must be deterministic. No wall-clock time, no unseeded randomness, no dependence on file ordering or thread scheduling.
- Use table-driven tests for wide, uniform surfaces such as opcode coverage.
- Every bug fix gets a regression test that fails before the fix.
- `unwrap()` and `expect()` are correct in tests. A failed setup should abort the test immediately.
- Use property tests (`proptest`) for round-trips, parsers, and anything with an algebraic law.
- Fuzz every parser and every function that reads untrusted bytes (`cargo-fuzz`).
- Snapshot tests (`insta`) suit large structured output. Review every snapshot diff; never accept in bulk.
- Test public API through the public API. Reaching into private state couples the test to the implementation.

# 5. Performance

- Measure first. "Faster" without a number is a guess.
- Benchmark with `criterion`, on the release profile, with a fixed input set.
- Profile before optimizing, then fix what the profile shows rather than what looks slow.
- Work in this order: algorithm, data layout, allocation, branching. Reach for `unsafe` after all four.
- Prefer clarity in logic that must stay auditable, even at a cost. A cycle-accurate loop is read more often than it is run.
- Apply `#[inline]` only across a crate boundary and only with a measurement. Within a crate the compiler decides better.
- Configure the release profile deliberately: `lto`, `codegen-units = 1`, and `panic` behavior are project decisions, not defaults to inherit.

# 6. Dependencies

- Keep dependencies few. Do not add a crate for what `std` does in ten lines.
- Check what the project already depends on before adding something new.
- Judge a crate on maintenance history, transitive weight, and unsafe surface.
- Declare a `rust-version` (MSRV) and enforce it in CI. Raising it is a deliberate change.
- Commit `Cargo.lock` and build CI with `--locked`, so a transitive release cannot break a build silently.
- Run `cargo deny check` for advisories, duplicate versions, and license policy. Run `cargo audit` if `deny` is not configured.
- Keep features additive. Cargo unifies features across the dependency graph, so a feature that removes an API or conflicts with another feature will break a consumer.
- Review dependency updates on a schedule rather than when something breaks.

# 7. Lints and CI

- Enable in CI, as required checks:
  - `cargo fmt --check`
  - `cargo clippy --all-targets --all-features -- -D warnings -D clippy::all -D clippy::pedantic`
  - `cargo test --all-targets --all-features --locked`
- Allow a pedantic lint narrowly, at the smallest scope, with a stated reason: `#[allow(clippy::cast_possible_truncation, reason = "PC wraps at 16 bits by design")]`.
- Prefer `#[expect(...)]` to `#[allow(...)]`. It fires when the lint stops triggering, so stale exemptions get deleted instead of accumulating.
- Deny `clippy::todo`, `clippy::unimplemented`, `clippy::dbg_macro`, and `clippy::unwrap_used` in production code, allowing the last in `#[cfg(test)]`.
- Run `cargo doc --no-deps` with warnings denied, so broken intra-doc links fail the build.

# 8. What Relaxes in Tests

- `unwrap()`, `expect()`, and `panic!` are correct in tests, benches, examples, and test-only helpers.
- Cloning for convenience is fine.
- The rest of this document applies unchanged. Test code is read as often as production code.

# 9. Review Checklist

- No `unwrap`, `expect`, or reachable panic in production paths.
- Errors are typed, propagated, and carry context.
- Public API is minimal, documented, and `#[non_exhaustive]` where it should be.
- Casts are `From`/`TryFrom`, or `as` with a stated reason.
- Arithmetic that can overflow says which behavior it wants.
- Behavior-changing fixes ship with a regression test.
- No `allow` without a reason, no `unsafe` without a `// SAFETY:` comment and a benchmark.
- Determinism preserved.
