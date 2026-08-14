# 1. Coding and Style
- Write clean, surgical, unambiguous, modular code. Over-engineering is not a benefit. Simplicity is the ultimate sophistication.
- Prioritize readability over cleverness.
- Match the surrounding code. Its naming, structure, and comment density are the local standard, even where they differ from these rules.
- Keep changes scoped to the task. Don't refactor unrelated code, rename things in passing, or fix unrelated style while you're in the file.
- Defer to the project's formatter and linter. Never hand-align, and never fight the tool.
- One concept per file. Split a file when it stops having a single describable job.

## 1.1 Naming
- Name for what the thing is or does. `data`, `info`, `manager`, `helper`, `handler`, and `util` say nothing.
- Don't invent abbreviations. Use one only if the codebase already uses it.
- Length should track scope. A loop index can be `i`; a module-level export cannot.

## 1.2 Comments
- Comment only what needs explanation. Lines that read plainly need nothing.
- Comment intent, not mechanics. Say why the code does this, not what it does.
- Don't narrate the change. No "added null check", no "moved from utils", no dated notes. The diff and the history already record that.
- Don't write a novel. A clause usually beats a sentence, and a sentence usually beats a paragraph.

## 1.3 Documentation
- Include documentation where a reader would otherwise guess: public interfaces, non-obvious constraints, and anything with a surprising failure mode.
- Keep it short and specific. Skip it entirely when the signature already says everything.

## 1.4 Abstraction and Dependencies
- Don't abstract for a single caller. No interface, factory, wrapper, or config knob until a second caller exists.
- Prefer the standard library. Don't add a dependency for something you can write in ten lines.
- Check what the project already depends on before reaching for something new.
- Delete dead code. Don't comment it out and don't keep it "for reference" — that is what history is for.

## 1.5 Errors
- Fail loudly. An error that disappears is worse than a crash.
- No bare `except:` or empty `catch {}`. Catch the specific failure you expect to handle.
- Never swallow an error to make a test or a build pass.
- Don't wrap code in defensive try/catch when it cannot throw.
- Check what you get back. Don't assume a call succeeded because it returned.

# 2. Correctness
- Don't claim code works without running it. Say what you ran, and say what failed.
- Report failures plainly. A skipped step, a broken test, or an unverified assumption gets stated, not omitted.
- Don't optimize without a measurement. "Faster" is a claim, and it needs a number.

## 2.1 Tests
- Test behavior, not implementation. A test that breaks on a refactor with no behavior change is testing the wrong thing.
- Don't mock code you own. Mock the boundary, not the internals.
- Don't assert on log strings.

# 3. Design Documents
- Keep prose strictly factual and objective.
- Keep sections reasonably sized. Cut bloat.
- Follow [writing.md](writing.md) for prose style. It covers filler, AI-isms, and terms like "what matters".
