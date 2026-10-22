---
name: object-calisthenics
description: Audit a C# file or changeset against Jeff Bay's 9 Object Calisthenics rules and propose or apply refactors one rule at a time. Use when the user mentions object calisthenics, asks to tighten up OO design, reduce nesting or primitive obsession, or wants a calisthenics pass on a file, class, or git diff.
---

# Object Calisthenics

Nine constraints that force small, behaviour-rich objects. They are an exercise, not law: apply them where they improve the code, and say so when a rule doesn't fit.

## Workflow

1. **Scope.** Target = file/class named by the user, else `git diff master...HEAD` (changed lines only, don't audit untouched code).
2. **Skip exempt types.** EF entities, DTOs, GraphQL/gRPC/proto types, options classes (`IOptions<T>`), records used as data carriers. Rules 3, 8, 9 don't apply to them. Say what was skipped.
3. **Scan** against the 9 rules below. Read [REFERENCE.md](REFERENCE.md) for the C# smell/fix pairs.
4. **Report** a table: `rule | location (file:line) | smell | proposed fix | effort (S/M/L)`. Rank by payoff, not rule number. Cap at ~10 findings; note the rest.
5. **Stop and ask which to apply.** Do not edit before the user picks (they want the approach explained first).
6. **Apply one rule at a time**, smallest first. After each: `dotnet build` and the relevant `.UT` tests. Never mix a calisthenics refactor with behaviour changes.
7. **Finish** with `dotnet csharpier format` on touched files and a summary of rules applied vs deliberately declined.

## The 9 rules

1. **One level of indentation per method.** Extract the inner block into a named method; use guard clauses and early `continue`.
2. **Don't use `else`.** Early return, guard clauses, polymorphism, or a lookup. A plain `if/else` assigning one value is fine as a ternary.
3. **Wrap all primitives and strings** that carry domain meaning (`OrderId`, `ApproverName`) in a small value type (`readonly record struct`). Don't wrap `bool`/loop counters/`CancellationToken`.
4. **First-class collections.** A class holding a collection holds nothing else. Put filtering/lookup behaviour on the wrapper instead of scattering LINQ.
5. **One dot per line.** Don't reach through objects (`order.Customer.Address.City`); add a method that answers the question (Law of Demeter). Fluent builders, LINQ chains and `Task` / `?.` chains are exempt.
6. **Don't abbreviate.** Names a full word or two. If the name is long, the method is doing too much, or the class is misplaced. Allow `id`, `db`, `ct`, lambda `x` of 1 line.
7. **Keep entities small.** Target: methods <= ~10 lines, classes <= ~50 lines, <= ~10 types per namespace. Treat as a smell threshold, not a hard fail.
8. **No more than two instance variables per class.** Injected dependencies count. Three+ means split the class or group related fields into a value object.
9. **No getters/setters/properties exposing state.** Tell, don't ask: move the logic that reads state into the owning object. Exempt: data carriers (see step 2).

## Judgement calls

- Rules 3, 8, 9 conflict most with EF/DI-heavy code. Flag, don't force.
- Rule 5 and LINQ: never "fix" a LINQ chain.
- A refactor that adds more code than it removes needs a clear readability win; otherwise decline and say why.
