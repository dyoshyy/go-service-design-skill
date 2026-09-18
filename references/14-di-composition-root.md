# Ch. 14: Where to write the dependency wiring

## The book's default
Concentrate dependency wiring in a single composition root (`main`, or an initialization-only package directly under `main`), and start with manual DI.
- The composition root is the outermost layer, so it is the one place allowed to import everything from the domain layer through the infrastructure layer. This is not an exception to the dependency rule; it is the role of the outermost layer itself.
- If wiring is scattered across handlers and Interactors, you can no longer trace "which implementation is used where". With one location, there is also only one place to swap implementations.
- Manual DI is nothing more than calling constructor functions in dependency order. Type mismatches become compile errors, the initialization order is visible by reading top to bottom, and there is no tool to learn.
- Interface-satisfaction checks like `var _ usecase.Xxx = (*postgres.Xxx)(nil)` also belong here, since they are wiring declarations: "use this implementation as this interface".
- When it grows too large, extract per-module assembly functions (e.g. `internal/order/di.NewOrderHandler(db, mailer)`). `main` is then reduced to creating cross-cutting resources (DB connection, config, logger) and calling the assembly functions.
- Recommended progression: manual DI → per-module assembly functions → wire, only if wiring changes still become a source of waiting time.

## When it applies, and exceptions
- For small-to-medium projects, or projects that have just started their design, manual DI is the only choice. It works fine without any additional tooling.
- Once wiring exceeds several hundred lines and fixing up arguments every time a constructor is added becomes a bottleneck, consider google/wire. The generated output is ordinary Go code, and type mismatches are caught at compile time.
- wire requires writing each interface-to-implementation mapping one at a time with `wire.Bind`, which fits poorly with the style of defining many small interfaces on the consumer side (the Binds pile up).
- Backing out of wire only requires committing the generated code, so adopting it is reversible.
- uber-go/dig and fx resolve dependencies via runtime reflection, so wiring mistakes become startup errors. Do not choose them unless there is a clear reason to give up compile-time verification (e.g. a large organization standardized on fx).

## How to spot violations
- A handler or Interactor directly news up a concrete implementation, such as `postgres.NewXxxRepository(db)` (wiring has leaked outside the composition root).
- A package other than `main` imports both the infrastructure layer and the interface layer.
- `var _ Interface = (*Impl)(nil)` is scattered all over the infrastructure layer, so which interface an implementation is used as is written somewhere other than the wiring.
- `main` is several hundred lines long and has not been split into per-module assembly functions.
- There are DI container `Provide` / `Invoke` calls, and dependency resolution failures only surface at startup rather than in tests.

## Source
- Ch. 14「DI とコンポジションルート〜依存の配線をどこに書くか〜」(DI and the composition root: where to write the dependency wiring) https://github.com/135yshr/documents/blob/main/books/go-service-design/di-composition-root.md
