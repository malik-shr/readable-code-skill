---
name: readable-code
description: Write readable, pragmatic, compact production code that feels like it was written by an experienced developer — not over-abstracted, over-commented, over-defensive AI code. Covers readability-first decision making, following the existing codebase, keeping diffs scoped, explicit-over-clever, pragmatic DRY and abstraction, flat control flow, sparse English comments, validation at trust boundaries, meaningful error handling, SQL-first data access, and a concrete list of AI-style overengineering to avoid. Use this skill whenever you are about to write, modify, extend, or refactor code in this project — including small edits, bug fixes, and single functions — not only when the user explicitly asks for "clean", "simple", or "readable" code. Read it before writing the first line, since almost all of it shapes decisions made while writing rather than afterwards.
---

# Readable Code

Readability is the highest priority. Every other rule here exists to serve it,
and when two guidelines conflict, the one that leaves the next reader with less
to figure out wins.

Picture that reader concretely: a competent developer who has never seen this
file, is opening it because something broke, and has about thirty seconds of
patience. They should be able to read top to bottom and understand what happens
without holding a diagram in their head or chasing one idea through five files.

Prefer boring, explicit, predictable code. Do not optimize for the fewest
lines. Do not try to make code look professional through extra abstractions,
layers, or patterns — make it easy for a professional to understand, which is
usually the opposite thing.

## Decision priority

Correctness is not on this list — it is the precondition. Code that does the
wrong thing beautifully is not a candidate. Among solutions that are correct:

1. **Readability** — how fast does someone new understand this?
2. **Simplicity** — fewer moving parts, fewer concepts held at once
3. **Consistency with the existing codebase** — matching surrounding conventions
4. **Maintainability** — how easy is this to change later?
5. **Performance** — unless it is genuinely critical for this task, in which
   case it moves up and you say so explicitly

## When an instruction overrides this document

Everything here is a default for decisions nobody has made explicitly. A direct
instruction from the user — build a wrapper here, split this into an interface,
keep that try/catch — outranks all of it, and is not something to argue about
or quietly skip. The user knows constraints this document does not.

If an instruction does seem to work against readability, implement what was
asked and say once, briefly, what concerns you. Do not repeat it, and do not
build a compromise version that satisfies neither the instruction nor the
guideline.

## What this document does not decide

Test strategy is out of scope here: whether something gets a test, what that
test covers, and in which order code and test are written are decisions this
document does not make. Test code is code, so everything below applies to it as
well.

## Start with the existing codebase

Before implementing anything, look at the surrounding code and at similar
implementations elsewhere in the repository. Most decisions this document
covers have already been made somewhere in the project, and matching that
answer is usually better than deriving a new one.

Follow existing reasonable conventions rather than introducing a different
pattern because it is a generic best practice. Generic best practices do not
automatically override reasonable project conventions — an unfamiliar "better"
style makes the file as a whole harder to read, which costs more than it gains.

Reuse the libraries, utilities, and abstractions already in the project when
they fit. Do not introduce a second way of solving a problem that already has
an established solution; two competing approaches are worse than either one
alone, because now every reader has to learn both and every future change has
to pick a side.

If you do deviate from a local convention, say why.

## Keep the change scoped

Keep changes focused on the requested task. Do not do unrelated refactoring, and
do not rename, reorganize, or abstract surrounding code unless the requested
change actually requires it.

A large diff costs review attention and buries the actual change among noise, so
growing it needs a concrete benefit. Improvements you notice but do not make are
worth mentioning to the user rather than silently performing.

Do not solve hypothetical future requirements. Build what was asked for.

When a requirement is genuinely ambiguous — may this field be empty, what
happens on a duplicate, is this list ever unbounded — do not settle it by
inventing a rule and quietly encoding it. Ask when the answer changes the shape
of the code; otherwise take the obvious reading, implement it, and state the
assumption in one line when reporting back. An invented rule buried in an
implementation is expensive to find later, because nothing marks it as a guess.

## Explicit over clever

Choose the implementation that reads most easily, not the shortest or most
elegant one. Density is not a virtue; the person debugging this at 2am is not
impressed by how few lines it took.

Prefer plain, visible constructs — conditionals, loops, named variables, early
returns — and avoid magic. Magic is anything whose behavior is not visible at
the place it happens: metaprogramming, reflection, dynamically resolved
attributes or methods, operator overloading that does something unexpected,
implicit type coercion, decorators or hooks that silently change semantics,
action at a distance through shared mutable or global state.

The objection to magic is not that it is advanced. It is that the reader cannot
find the behavior by reading the code in front of them — they have to already
know it is there. That turns every future change into an archaeology exercise.

Idiomatic language features are not magic. Comprehensions, pattern matching,
context managers, iterators, destructuring, and the standard library are how a
fluent reader of that language expects code to look; avoiding them produces
code that is longer and stranger, not clearer. The test is whether a competent
developer in that language sees what happens by reading it — not whether the
construct would exist in some other language.

```
// dense — three levels to unpack before you know what it returns
status = isActive ? (isPaid ? "active" : "grace") : (wasPaid ? "expired" : "new")

// explicit — obvious at a glance, and each branch has room to grow
if (!isActive) {
    return wasPaid ? "expired" : "new"
}
return isPaid ? "active" : "grace"
```

Avoid long chains of map/filter/reduce when a straightforward loop communicates
the algorithm more clearly. A chain that transforms one thing is fine; a chain
that hides the actual logic in four stacked lambdas is not.

## Compact without being clever

Express the required behavior with as little structural overhead as reasonably
possible. More code is not more robust or more professional. Avoid unnecessary
boilerplate, intermediate variables, helper functions, wrappers, abstractions,
and defensive checks — every function, type, class, variable, or abstraction
should earn its place with a concrete readability or architectural benefit.

Specifically:

- Do not split straightforward logic across several functions or files when
  keeping it together is easier to understand.
- Prefer a direct implementation when an abstraction would only add indirection.
- Do not add layers that merely forward data or calls without adding behavior.
- Do not create a variable that just renames another value without clarifying it.
- Do not write a helper for a trivial one-time operation.
- Avoid transformations between structurally similar data types.
- Keep the number of concepts the reader must hold as small as possible.

Compact does **not** mean clever or compressed. Do not reduce line count with
dense one-liners, nested ternaries, complex boolean expressions, excessive
method chaining, obscure language features, or by combining unrelated
operations. A slightly longer implementation wins whenever it is substantially
easier to read.

When two implementations are equally readable and correct, prefer the one with
fewer concepts, then fewer abstractions, then fewer layers, then fewer
branches, then less code. The goal is not minimum line count — it is minimum
unnecessary code.

## Abstractions and DRY

Avoid premature abstraction. Do not create helpers, wrappers, interfaces,
factories, services, or generic utilities without a concrete benefit, and
follow DRY pragmatically rather than dogmatically.

Similar-looking code does not automatically need abstraction. Shape is not
shared meaning: code that looks alike today often diverges tomorrow for
unrelated reasons, and then the shared helper grows flags and branches to serve
both callers badly. A small amount of obvious duplication is preferable to the
wrong abstraction — duplication is cheap to fix later, a bad abstraction is not.

Extract when the extracted piece has a meaningful responsibility, a name that
means something on its own, real reuse, or when it genuinely improves
readability by hiding uninteresting detail.

Avoid excessive fragmentation. Splitting a coherent thirty-line function into
six five-line functions each used once makes the pieces look tidy while the
whole becomes unreadable, because understanding the operation now means
reassembling it from fragments. A readable thirty-line function is often the
better answer.

## Control flow

Keep control flow simple and flat. Prefer guard clauses and early returns where
they reduce nesting — deep nesting is one of the strongest readability killers,
and handling edge cases up front keeps the main path at the outermost level.

Avoid large boolean expressions that combine many unrelated checks. When a
business condition is genuinely complex, give it a meaningful name so the
condition reads as a statement about the domain rather than a puzzle. But do
not extract trivial conditions just to satisfy the rule — `if (user.isActive)`
does not need a wrapper.

## Side effects and state

A function should do what its name says and not much else. Hidden side effects
— a "get" that writes, a validator that mutates its input, a helper that
refreshes a cache on the way past — are among the hardest bugs to find, because
the code causing the problem does not look like it does anything.

Keep side effects at the edges and visible in the name. Prefer returning a new
value over mutating an argument, and prefer passing state explicitly over
reaching into shared or global state. This is not a push toward functional
purity for its own sake — mutation is often the clearest thing to write. The
requirement is only that the reader can see where it happens.

## Naming

Naming is the highest-leverage readability tool available and the cheapest to
get right while writing. A good name removes the need for a comment, for an
extracted variable, and often for the abstraction itself.

Name things after what they mean in the domain, not after their type,
structure, or mechanics. `data`, `result`, `item`, `value`, `info`, `handle`,
`process`, `manage`, and `doWork` describe nothing — they are what you reach
for before deciding what the thing actually is. When a precise name is hard to
find, that is usually a design signal rather than a vocabulary problem: the
thing probably has more than one responsibility.

Let length follow scope. A loop counter can be `i`; something exported from a
module should say what it is. Booleans read as claims (`isExpired`,
`hasAccess`), not as questions or negations — `notReady` forces the reader to
invert it at every use. Keep one word per concept across the codebase: if the
project says `customer`, do not introduce `client` for the same thing.

Name a literal value when the value itself does not explain the rule.
`if (attempts > 3)` is perfectly clear inside a short function about retries;
`MAX_LOGIN_ATTEMPTS` earns its name once the number appears in several places
or its meaning is not obvious from context.

## Comments

Models write far too many comments. Treat the urge to add one as suspect until
you can name the thing the reader would have failed to understand without it.

Comments are English, sparse, short, and precise. They explain **why** something
surprising or non-obvious exists — not what the next line does.

```
// increment the retry counter          <- deletable, the code says this
retryCount = retryCount + 1

// The vendor returns 200 with an empty body when rate-limited, so an empty
// response is retried instead of being treated as "no results".
if (response.body.isEmpty()) { ... }
```

The test is simple: cover the comment with your hand. If the code below still
tells you the same thing, delete the comment.

Before writing a comment, try to make it unnecessary. A better name, a
well-named intermediate value, or a flatter structure replaces a comment more
often than not — and unlike a comment, a name cannot drift out of sync with the
code.

Do not automatically add documentation comments to every function, class, or
property. A docstring repeating the function name and the parameters already
visible in the signature is noise. The exception is consistency: if a module
documents its functions throughout, match it rather than leaving one bare
function among documented neighbours.

## Validation

Validate at trust boundaries — where data enters the system — and trust it
afterwards. Trust boundaries are user input, HTTP requests, responses from
external APIs, environment variables, and files or other external data.

Prefer the schema-based validation library the project already uses (Zod,
Valibot, Joi, Yup, Pydantic, or an equivalent) over hand-assembling type checks,
null checks, regexes, and length checks at the call site. Do not introduce a new
validation dependency when the project already has an established approach.

```
// inline and scattered — the actual rule is hard to see
if (value !== null && value !== undefined && typeof value === "string"
    && value.length >= 3 && value.length <= 50
    && /^[a-zA-Z0-9_-]+$/.test(value)) { ... }

// stated once, in one place, reusable and readable
usernameSchema = string().min(3).max(50).matches(USERNAME_PATTERN)
```

Once data has passed validation, downstream code operates on the validated
types and invariants. Re-checking the same property at several layers without a
concrete reason adds noise and signals to the reader that the guarantee is not
real.

**Do not overvalidate.** Every validation rule must correspond to a real
requirement. Do not add regex restrictions, length limits, null checks, type
checks, fallback values, or sanitization just to look defensive. Invented
constraints are worse than absent ones — they reject valid data, and the next
developer cannot tell which rules are real.

**Prefer contracts over defensive checks.** When a value is already guaranteed
by schema validation, the type system, a database constraint, a framework
guarantee, or an established internal contract, do not keep defending against
violations of it without a realistic reason to expect one. Validate untrusted
data once, establish a trusted representation, and keep everything after that
boundary simple.

## Error handling

Handle errors where something meaningful can actually be done about them.
Everywhere else, let them propagate — a stack trace that reaches the top
intact is more useful than one interrupted by handlers that added nothing.

Avoid catch-and-rethrow that adds no information or behavior, and in particular
do not add a try/catch merely to log an error and throw it again. That pattern
produces duplicate log noise and hides the real handler.

Do not conceal unexpected states behind fallback values. Required data that is
missing should fail clearly rather than being replaced with an arbitrary
default, because a silent default turns a loud bug into a quiet one that
surfaces much later and much further away.

Distinguish expected errors that are part of normal application behavior — a
validation failure, a not-found lookup, a declined payment — from unexpected
programming or system errors. The first kind is control flow and belongs in the
design; the second kind should generally be allowed to fail.

Catch only errors you can name. A `try/catch` should exist because a specific
thing realistically goes wrong there — the network times out, a unique
constraint is violated, an external source returns something that is not valid
JSON. Wrapping code that only throws when the program itself is wrong swallows
exactly the bug you need to see.

Do not build error infrastructure ahead of need. Custom error hierarchies,
result wrappers, error codes, retry and backoff logic are answers to a caller
that exists and needs them. Until then they are structure around a problem
nobody has had yet.

## SQL first

Prefer SQL for operations databases naturally handle well: filtering, sorting,
joins, grouping, aggregation, existence checks, deduplication, and pagination.
Select only the data actually required where practical, and avoid loading rows
into application code just to process them there.

```
// loads every order into memory to answer a question SQL could have answered
orders = db.query("SELECT * FROM orders")
recent = orders.filter(o => o.customerId == id && o.createdAt > cutoff)
total = sum(recent.map(o => o.amount))

// one round trip, one row back
total = db.queryOne(
    "SELECT COALESCE(SUM(amount), 0) FROM orders
     WHERE customer_id = ? AND created_at > ?",
    id, cutoff
)
```

Do not force complex business logic into SQL when the SQL version would be
significantly harder to understand. An intricate rule that needs branching,
external lookups, or explanation usually reads better in application code.

SQL first means using the database for database-shaped problems, not maximizing
the amount of SQL.

## Avoid AI-style overengineering

Do not add things because they could theoretically be useful. Each construct
should solve a concrete, present problem. The recurring offenders:

fallback values · null checks · type checks · regex validation · optional
chaining · try/catch blocks · helper functions · wrapper classes · interfaces ·
DTOs · mappers · factories · configuration options · extension points ·
comments · compatibility layers · handling of hypothetical edge cases

None of these are forbidden — each is right sometimes. The failure mode is
adding them reflexively, as a substitute for knowing whether they are needed.
If you cannot name the concrete problem a construct solves, remove it.

## What this looks like in practice

The same endpoint written both ways, in illustrative pseudocode. The first
version is what a model produces by default. Every construct in it is
individually defensible, which is exactly why they accumulate.

```
interface OrderSummaryDTO {
    customerId: string
    total: number
    count: number
}

class OrderSummaryMapper {
    static toDTO(raw) {
        return {
            customerId: raw?.customerId ?? "",
            total: raw?.total ?? 0,
            count: raw?.count ?? 0,
        }
    }
}

/**
 * Gets the order summary for a customer.
 * @param customerId The customer id
 * @param since The cutoff date
 * @returns The order summary DTO
 */
function getOrderSummary(customerId, since) {
    // validate the input
    if (!customerId || typeof customerId !== "string" || customerId.length === 0) {
        throw new Error("Invalid customer id")
    }

    try {
        // fetch all orders from the database
        orders = db.query("SELECT * FROM orders")

        // filter the orders for this customer
        customerOrders = orders.filter(o => o?.customerId === customerId)

        // filter by date
        recent = customerOrders.filter(o => o?.createdAt > since)

        // calculate the total
        total = recent.reduce((sum, o) => sum + (o?.amount ?? 0), 0)

        return OrderSummaryMapper.toDTO({ customerId, total, count: recent.length })
    } catch (error) {
        logger.error("Failed to get order summary", error)
        throw error
    }
}
```

The same behavior, written to be read:

```
function getOrderSummary(customerId, since) {
    return db.queryOne(
        "SELECT COALESCE(SUM(amount), 0) AS total, COUNT(*) AS count
         FROM orders
         WHERE customer_id = ? AND created_at > ?",
        customerId, since
    )
}
```

What went, and why:

- **The interface, the DTO, and the mapper** named no concept the query result
  did not already carry. They were three indirections that renamed a row.
- **Filtering, summing, and counting** moved into SQL. The original loaded the
  entire orders table to answer a question about one customer.
- **The input check** disappeared because `customerId` arrives already
  validated from the request schema at the boundary. Re-checking it here told
  the reader the guarantee was not real.
- **The optional chaining and `?? 0` fallbacks** defended against rows the
  database schema does not permit — and would have converted a genuinely broken
  row into a silently wrong total, which is worse than an error.
- **The try/catch** logged and rethrew: a duplicate log line and no behavior.
- **Every comment** narrated the line beneath it, and the docstring restated the
  signature.

The lesson is not that shorter is better. If this summary involved a real
business rule — tiered discounts, currency conversion, partial refunds — that
rule would belong in application code and the function would legitimately be
longer. What shrank here was overhead, not logic.

## Final self-review

This is an internal pass. The point is to catch these things before the
user sees them, not to report the checklist back — fix what needs fixing
and stay quiet about the rest.

Before finishing, read the complete diff and ask:

- Is this the simplest readable solution?
- Did I introduce unnecessary abstractions?
- Did I add defensive checks for impossible or unrealistic states, or catch an
  error whose realistic cause I cannot name?
- Did I invent requirements or edge cases?
- Could some application-side data processing be expressed more naturally in SQL?
- Are there comments that merely repeat the code?
- Did I add fallback behavior that could hide a bug?
- Did I change anything unrelated to the task?
- Does the implementation match the surrounding codebase?
- Could any clever construct be replaced by something more obvious?

Simplify where it improves readability. But do not refactor merely to satisfy
these guidelines — that would violate the scope rule above, and the point of
this document is better code, not compliance with it.
