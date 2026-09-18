# Ch. 13: How to split packages

## The book's default
Cut directories per bounded context, and place a nested internal in each context to protect its internal implementation.
The same "product" needs different attributes in catalog, inventory, and order. One shared Product makes everyone carry attributes no context needs.
Each context has its own domain layer / UseCase layer / infrastructure layer, with distinct model names (Product and Stock are separate types).
A single top-level internal/ cannot prevent imports between contexts, because the parent directory is shared.
Nesting as `<ctx>/internal/` makes imports from other contexts a compile error.
infra, the public event schema (event/), and ports (port/) go outside the nested internal, and integration passes through them.
The integration interface is owned by the consuming context and implemented by the providing context's adapter (dependency inversion).
Events are owned by the publisher; only things whose infrastructure implementation is shared by all contexts, such as EventPublisher, go in shared.

```text
internal/
  order/
    internal/{domain,usecase}/   # importable only from order
    event/                       # published events (public)
    port/                        # interfaces owned by the consumer
    infra/postgres/
  inventory/
    internal/{domain,usecase}/
    adapter/                     # implements order/port
    infra/postgres/
pkg/shared/                      # value objects such as Money, EventPublisher
```

## When it applies, and exceptions
- Adopt when: multiple contexts live in one repository and the project is deployed as a monolith
- Consider multi-module setup with Go Workspaces when teams are separate and development cycles and deployments are independent. The adoption cost is somewhat high
- Even without go.work, simultaneous editing is possible via replace in each go.mod. The advantage of Workspaces is not having to write replace, and putting go.work in .gitignore gives each developer freedom
- The only things allowed in shared are value objects such as Money and infrastructure interfaces for which every context is injected with the same implementation. Domain types do not go in
- When referring to another context's entity, hold only the ID string; do not hold the other side's struct
- Synchronous integration via shared interfaces (beware of TOCTOU) and asynchronous integration via events (beware of Dual Write) may coexist in the same Interactor. Compensate the former with optimistic locking and the latter with the Outbox

## How to spot violations
- Another context's `.../domain` or `.../usecase` is imported directly. Without a nested internal this goes unnoticed (supplement with depguard)
- There is exactly one Product struct carrying the concerns of every context
- Domain models and repository interfaces keep accumulating in the shared package
- The interface used by the consumer is defined in the provider's context
- Event type names are Subscribed with magic strings

## Source
- Ch. 13 「モジュール構成〜境界づけられたコンテキストを Go に落とし込む〜」 (Module structure: mapping bounded contexts onto Go) https://github.com/135yshr/documents/blob/main/books/go-service-design/module-structure.md
