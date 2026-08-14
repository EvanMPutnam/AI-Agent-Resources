# 1. NASA Flight Software Rules

Gerard Holzmann wrote the Power of 10 at NASA's Jet Propulsion Laboratory in 2006, for C code that runs on spacecraft. The rules are stricter than anything in [coding-rules.md](coding-rules.md), and some of them contradict it.

This file is not global. A project adopts it whole or not at all. If there are competing rules, this is authoritative.

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

