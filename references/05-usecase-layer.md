# Ch. 5: Can the UseCase layer be omitted?

## The book's default
Put a UseCase layer on every feature. Don't omit it even when it is a pass-through that only calls the Repository.
Pass-throughs don't stay pass-throughs for long. Idempotency checks (return the existing result if already analyzed), mutual exclusion (reject if an import is already running), and combining multiple strategies and merging their results were all logic that later entered Interactors that had been pass-throughs; without a UseCase layer they leak into Handlers in the interface layer.
In batch processing, "keep going when one item fails" and "tally success/failure counts and elapsed time" belong to neither the domain layer nor the infrastructure layer; application-specific logic needs a place to live.
If the Handler defines the interface, dependency inversion holds even without a UseCase layer, but then the Handler knows the structure of domain-layer Entities directly. With a UseCase layer in between, the Handler depends only on Input/Output DTOs, and domain-model changes don't ripple into the interface layer.
When some features have a UseCase layer and others don't, cross-cutting changes such as authorization checks take longer to locate. The uniform rule "business logic is always in the UseCase layer" is conceptually simple, and a pass-through is not a sign of a problem but headroom for logic to be added later.

## When it applies, and exceptions
- It pays off most in projects centered on analysis, batch processing, and external-service integration. It becomes the place where idempotency, mutual exclusion, multiple strategies, and tolerance of partial failure accumulate
- In a simple CRUD-centric application, many Interactors will remain pass-throughs. There it is more reasonable to introduce the layer incrementally when it becomes necessary; with implicit interfaces, the refactoring to insert it later is feasible
- Standardizing from the start is cheaper than introducing it midway for cross-cutting changes that touch multiple features at once
- The testing burden barely grows. Mock the one- or two-method private interface defined on the Handler side, and having a test file already on the UseCase side lowers the bar to writing tests when logic is added
- Interactor complexity may range from "Repository call only" to "idempotency + multiple strategies + merge". What is standardized is presence, not thickness

## How to spot violations
- A Handler holds a Repository interface directly as a field, calls `FindByID`, and converts to JSON
- A Handler handles domain-layer types such as `*model.User` directly, doing response conversion or branching
- Idempotency checks, mutual exclusion, authorization checks, and cache decisions are written in Handlers or middleware and duplicated across multiple Handlers
- Within one service, features with a UseCase layer and features without one are mixed
- Tallying partial failures in batch processing or assembling error lists is written in a Handler or the infrastructure layer

## Source
- Ch. 5 「UseCase 層は必要か〜パススルーの価値を考える〜」 (Is the UseCase layer necessary? On the value of pass-throughs) https://github.com/135yshr/documents/blob/main/books/go-service-design/usecase-layer.md
