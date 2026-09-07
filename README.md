# readable-code-skill

A Claude Code plugin that makes Claude write readable, pragmatic, compact
production code — instead of the over-abstracted, over-commented,
over-defensive code models tend to produce by default.

It is language-agnostic and applies while code is being written, not as a
review step afterwards.

## What it enforces

- **Readability first** — among correct solutions, the one that is fastest to
  understand wins. Correctness is the precondition, not a tiebreaker.
- **Follow the existing codebase** — local conventions beat generic best
  practices; no second way to solve an already-solved problem.
- **Scoped diffs** — no unrelated refactoring, no solving hypothetical future
  requirements.
- **Explicit over clever** — no magic, no behavior that is invisible at the
  place it happens. Idiomatic language features are welcome.
- **Compact without being clever** — minimum unnecessary code, not minimum
  line count.
- **Pragmatic DRY** — small obvious duplication beats the wrong abstraction.
- **Flat control flow** — guard clauses and early returns over deep nesting.
- **Sparse comments** — English, short, explaining _why_, never narrating the
  next line. Docstrings count as comments.
- **Validation at trust boundaries** — schema-based, once, then trust the
  contract. No invented restrictions.
- **Meaningful error handling** — catch only errors with a realistic, nameable
  cause; no catch-and-rethrow, no silent fallbacks hiding bugs, no error
  infrastructure built ahead of need.
- **SQL first** — filtering, sorting, joins, grouping, aggregation and
  pagination belong in the database, unless the SQL version is harder to read.
- **No AI-style overengineering** — an explicit list of the constructs models
  add reflexively, and the rule that each must solve a concrete problem.

Test strategy is deliberately out of scope — whether to write a test and what it
covers is left to a dedicated testing or TDD skill, so the two can be combined
without overlapping. Test code itself is still code and follows everything above.

## Installation

Clone the repo into your personal skills directory. Claude Code auto-loads
anything there on the next session — no marketplace, no install command.

```
git clone https://github.com/malik-shr/readable-code-skill.git ~/.claude/skills/readable-code-skill
```

It then applies in every project. Update later with:

```
git -C ~/.claude/skills/readable-code-skill pull
```

### Single project only

If you want it in one repository rather than everywhere, copy the skill into
that project and commit it:

```
mkdir -p .claude/skills
cp -r ~/.claude/skills/readable-code/skills/readable-code-skill .claude/skills/
```

Everyone who opens the project gets it, without installing anything.

## Structure

```
.claude-plugin/
  plugin.json        plugin manifest
skills/
  readable-code/
    SKILL.md         the philosophy itself
```
