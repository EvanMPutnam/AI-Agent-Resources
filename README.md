# House Style

Writing and coding rules I hand to AI agents. `sync.sh` compiles them into one
`CLAUDE.md` and installs it at `~/.claude/CLAUDE.md`, where Claude Code reads it for
every session on this machine.

## Contents

| File | What it covers | Applies to |
|---|---|---|
| `common/writing.md` | Orwell's six rules, and the LLM prose tics to strip out | every project |
| `common/coding-rules.md` | Naming, comments, abstraction, errors, tests, design docs | every project |
| `common/NASA-flight-software.md` | Holzmann's Power of 10 | specific projects |

The first two are what `sync.sh` installs globally. The third is copied into a project by
hand when that project needs it.

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

Drop it in `common/` and add its name to `SOURCES` in `sync.sh`. Order in the array is
order in the output.

## Why NASA is not global

`writing.md` and `coding-rules.md` apply to everything I write, so they belong in the
global file. `NASA-flight-software.md` does not. Holzmann wrote the Power of 10 for C
that flies on spacecraft and cannot be patched after launch, and the rules cost more
than most projects can justify — no dynamic allocation, no recursion, two assertions per
function.

A project that does justify them adopts the file whole, by copying it into that
project's own `CLAUDE.md`. Taking a rule here and there defeats the point: the list works
because a static analyzer can check every rule.

## Editing

Edit the files in `common/`, then rerun `sync.sh`. The generated `CLAUDE.md` carries a
comment saying so, and the next sync overwrites anything you change there.
