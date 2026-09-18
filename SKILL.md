---
name: go-service-design
description: Use when designing a Go service or API from scratch, deciding its layer structure or package layout, choosing where interfaces / validation / errors / transactions live, reviewing a Go design or PR against clean-architecture and DDD criteria, or when a Go design question starts with "how many layers", "where do interfaces go", "do we need a UseCase layer", or "how far should we abstract the DB".
---

# Go service design: decision criteria

There is no "correct structure". The only thing to protect is the direction of dependencies; everything else is a judgment call that depends on the situation.
This skill holds those judgment criteria so they don't have to be re-derived every time. The defaults come from the book listed under Source, paraphrased and organized as a decision map.

## How to use

- **When designing**: work through the five-step design below. When a step raises a question, read the matching file from the decision map.
- **When reviewing**: apply the "How to spot violations" section of each reference file and call out any symptom you find.
- **Constraints that must hold on every diff** live in `rule.md`. Placed under `.claude/rules/`, it loads automatically only when Go files are touched. It carries no rationale; the rationale lives in `references/`.

## Five-step design

Don't write a thick design document. Decide only these five things. Anything not decided here (table definitions, caching, auth) is added later, when it becomes necessary.

1. **List use cases as verbs.** Write "a user can ..." lines and cap the first version at about five. This list becomes the set of Interactors.
2. **Decide the domain model and invariants.** Start with one aggregate. Put 3-5 rules into words and protect them with unexported fields and state changes through methods only. Create value objects only for values that carry validation or behavior.
3. **Decide layers and directories.** One bounded context means no module split. Keep a UseCase layer. Pick the repository abstraction level (level 1 when in doubt). Define interfaces on the consuming side.
4. **Decide the split of errors and validation.** Format checks on the outside, invariants in the domain layer. Define sentinel errors in the domain layer and keep a table mapping them to HTTP status codes in the handler.
5. **Design the boundary with external APIs.** Define a minimal port in the UseCase layer and translate in an anti-corruption layer in the infrastructure layer. Decide here what happens when the external service is down.

## Decision map

| What you're unsure about | Default | Details |
| --- | --- | --- |
| How many layers do we need | Any number. Only the direction of dependencies matters | [02](references/02-dependency-rule.md) |
| Where do interfaces go | On the consuming side, with only the methods it needs | [03](references/03-interface-design.md) |
| Which architecture should we adopt | Don't pick by name. Let scale set the granularity | [04](references/04-architecture-comparison.md) |
| Can we skip the UseCase layer | Keep it. Pass-through is where future logic will live | [05](references/05-usecase-layer.md) |
| How far do we abstract the DB | Pick one of three levels. Level 1 when in doubt | [06](references/06-repository-levels.md) |
| How do we split errors | Separate domain errors from infrastructure errors | [07](references/07-error-handling.md) |
| Where does validation go | Format on the outside, invariants in the domain layer | [08](references/08-validation-design.md) |
| How many value objects | Only those with validation or behavior | [09](references/09-value-objects.md) |
| How do we protect invariants | Unexported fields, changes through methods | [10](references/10-aggregate-invariants.md) |
| Aggregates are too coupled | Cut the coupling with domain events | [11](references/11-domain-events.md) |
| External API formats are leaking in | Translate in an anti-corruption layer | [12](references/12-anti-corruption-layer.md) |
| How do we split packages | By context first, then by layer inside each | [13](references/13-module-structure.md) |
| Where do we wire dependencies | In the composition root. Start with manual DI | [14](references/14-di-composition-root.md) |
| Where do we open transactions | Question the aggregate first. Use a TxManager only across aggregates | [15](references/15-transaction-management.md) |
| What do we test, and how much | Cover the domain layer thickly with fast tests | [16](references/16-testing-strategy.md) |
| How do we keep the rules from eroding | Make static analysis a required CI check | [17](references/17-enforce-dependency-rule.md) |

Every file has the same sections: The book's default / When it applies, and exceptions / How to spot violations / Source.

## How to grow it

When a project teaches you that a default is wrong, incomplete, or needs a condition, revise the reference file itself: adjust the default, add an exception, or add a symptom to "How to spot violations". Write it in general terms that hold for any project. Project-specific rationale and decision logs stay in the project.

## Terminology

Aligned with the book: domain layer (innermost: entities, value objects, repository interfaces) / UseCase layer (`usecase/`; the implementing struct is an Interactor) / interface layer (HTTP handlers and other outer boundaries) / infrastructure layer (DB, external APIs).
A project may use other names (application, presentation, ...); the mapping is the same.

## Source

135yshr, 『Go サービス設計』 (Go Service Design): https://github.com/135yshr/documents/tree/main/books/go-service-design
The source repository carries no license, so nothing is reproduced verbatim; everything here is paraphrased.
