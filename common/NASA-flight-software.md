# 1. NASA Flight Software Rules

Gerard Holzmann wrote the Power of 10 at NASA's Jet Propulsion Laboratory in 2006, for C code that runs on spacecraft. The rules are stricter than anything in [coding-rules.md](coding-rules.md), and some of them contradict it.

## 1.1 The Rules

1. Restrict control flow to simple constructs. No `goto`, no `setjmp`/`longjmp`, no recursion.
2. Give every loop a fixed upper bound that a tool can prove statically.
3. Do not allocate memory dynamically after initialization.
4. Keep every function short enough to print on one page, roughly 60 lines.
5. Average at least two assertions per function.
6. Declare data at the smallest possible scope.
7. Check the return value of every non-void function, and check parameter validity inside every function.
8. Limit the preprocessor to header includes and simple macros. No token pasting, no variadic macros, no recursive macros.
9. Allow at most one level of pointer dereference. No function pointers.
10. Compile with every warning enabled from the first day, at zero warnings, and run static analysis daily.

## 1.2 Why They Read That Way

Two constraints shape the list. Flight software cannot be patched after launch, so one defect costs the mission. And Holzmann required every rule to be checkable by a static analyzer, which is why the rules give fixed numbers instead of asking for judgment.

Neither constraint holds here. Our code ships continuously, and people read it more often than tools do. Holzmann expected the rules to feel draconian and argued that flight software earns the cost. Most code does not.

## 1.3 What Transfers

**Rule 2, bounded loops.** This is the one to steal. A `while True` around a model call, a retry, or a poll can hang the process and run up a bill at the same time. Every loop gets a bound, a timeout, or a max-iteration count.

**Rule 6, smallest scope.** Language-independent and already good practice.

**Rule 10, zero warnings.** Maps onto lint-clean and type-check-clean. The sharper half of the rule is the corollary: a suppression without a comment justifying it is a warning you hid.

## 1.4 What Transfers as a Smell

**Rule 4, function length.** Sixty lines is arbitrary outside C. The usable version: a function you cannot describe in one sentence does too much.

**Rule 7, checked returns.** Exceptions do part of this automatically. The remaining half — do not assume a call succeeded because it returned — is already in `coding-rules.md` § 1.5.

**Rule 9, pointer depth.** No pointers in Python or TypeScript. The surviving idea is `a.b.c.d.e` chains and deep dictionary access, which costs readability rather than safety.

## 1.5 What Does Not Transfer

**Rule 1** targets C hazards. `goto` and `setjmp` do not exist for us, and recursion over a tree of known depth is fine.

**Rule 3** assumes manual memory management. Garbage collection makes the literal rule empty. Its descendant is worth keeping in mind: bound anything that accumulates, such as caches, conversation history, and growing lists.

**Rule 5** is the one to reject. Assertion density exists because C has no other runtime checking. Adopting a quota would pad every function to hit a number, which contradicts `coding-rules.md` § 1. Types and boundary validation do the same job with less noise.

**Rule 8** has no direct analog. The closest cousin is heavy metaprogramming: decorators that rewrite behavior, monkeypatching, dynamic imports. Treat those with the same suspicion, since code that is not what it appears to be causes the same debugging problem.

## 1.6 What We Take

Bound every loop, outside of a runloop, and treat an unexplained suppression as a warning. The rest stays a reference.
