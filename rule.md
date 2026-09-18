---
paths:
  - "**/*.go"
---

# Go service design: what a diff must never break

Rationale and exceptions are in the `go-service-design` skill under `references/`. Read them when in doubt.

- **Dependencies point inward only.** The domain layer imports neither the UseCase / interface / infrastructure layers nor external modules. Layers at the same depth (for example infrastructure and interface) don't import each other. Only the composition root (`cmd/`) may know concrete implementations.
- **Define interfaces on the consuming side, with only the methods it needs.** Don't put interfaces in the implementing package. If one method is enough, use one method.
- **Don't skip the UseCase layer.** Handlers never call repositories directly. Even if it looks like pass-through, put an Interactor in between.
- **Don't mix repository abstraction levels.** At level 1, no SQL or ORM types appear in domain-layer interfaces. Write the contract in comments, such as "never returns `(nil, nil)`".
- **Classify and wrap errors.** Wrap domain sentinels with `%w` so `errors.Is` can find them. Don't return input errors and I/O errors in the same shape. Only the interface layer converts to HTTP status codes.
- **Split validation by place.** Format (JSON shape, required fields, length) in the interface layer. Invariants in the domain layer. Validation that needs I/O, such as master-data lookups, in the UseCase layer.
- **Wrap values that carry validation or a range in a struct and validate in the constructor.** A defined type like `type X float64` lets `X(1.5)` bypass validation, so don't use it for values that need checking. IDs and enums may stay named types. Pair an error-returning `NewX` with a test-only `MustX`.
- **Aggregate state changes only through methods on unexported fields.** If `record.Status = ...` compiles from outside the package, the design is broken.
- **Never bring external API response types inside the UseCase layer.** Translate them into your own port types in the infrastructure layer.
- **Wire dependencies in `main` (the composition root).** Don't call `New*` inside a layer to assemble concrete types.
- **Check dependency direction mechanically in CI.** When you add a layer, add it to the checker's allow list too.
