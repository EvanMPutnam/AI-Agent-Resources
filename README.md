# House Style

Writing and coding rules I hand to AI agents. `sync.sh` compiles them into one
`CLAUDE.md` and installs it at `~/.claude/CLAUDE.md`, where Claude Code reads it for
every session on this machine.

## Contents

`common/` is global. `sync.sh` compiles it into `~/.claude/CLAUDE.md`, and it applies to
everything.

| File | What it covers |
|---|---|
| `common/writing.md` | Orwell's six rules, and the LLM prose tics to strip out |
| `common/coding-rules.md` | Naming, comments, abstraction, errors, tests, design docs |

`custom/` is per-project. Nothing here is installed globally — a project that wants one of
these files copies it into that project's own `CLAUDE.md`.

| File | What it covers | Adopt it when |
|---|---|---|
| `custom/NASA-flight-software.md` | Holzmann's Power of 10 | The code cannot be patched after it ships, or one defect costs more than the rules do |
| `custom/rust-rules.md` | Errors, types, unsafe, concurrency, lints, CI gate | The project is written in Rust |

## Usage

```sh
./sync.sh
```

The script reads the files listed in its `SOURCES` array and writes them to
`~/.claude/CLAUDE.md`, joined by a horizontal rule. It backs up an existing file to
`CLAUDE.md.bak` and exits without writing when nothing changed.

Set `CLAUDE_HOME` to install somewhere else:

```sh
CLAUDE_HOME=/tmp/claude-test ./sync.sh
```

A missing source file stops the script instead of dropping a section, so renaming a
file in `common/` fails loudly.

## Adding a file

A rule that applies everywhere goes in `common/`, and its name goes in the `SOURCES`
array in `sync.sh`. Order in the array is order in the output.

A rule that applies to one language or one class of project goes in `custom/` and stays
out of `SOURCES`.

## Why custom rules stay out of the global file

A global file is read on every session, so anything in it has to be true everywhere.
`writing.md` and `coding-rules.md` qualify. The rest do not, for two different reasons.

`NASA-flight-software.md` is too expensive. Holzmann wrote the Power of 10 for C that
flies on spacecraft and cannot be patched after launch, and no dynamic allocation, no
recursion, and two assertions per function cost more than most projects can justify.

`rust-rules.md` is too specific. A rule about `unwrap()` is noise in a Python project,
and rules that never apply teach the model to skim the ones that do.

Each is adopted whole, by copying it into a project's own `CLAUDE.md`. Taking a rule
here and there defeats the point: both lists work because a tool can check every entry,
and a half-adopted list goes back to being a matter of opinion.

## Editing

Edit the files in `common/`, then rerun `sync.sh`. The generated `CLAUDE.md` carries a
comment saying so, and the next sync overwrites anything you change there.

Files in `custom/` are copied by hand, so a project holds a snapshot rather than a live
link.
