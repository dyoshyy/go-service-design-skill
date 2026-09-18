# Ch. 12: External API formats leaking in

## The book's default
Confine external systems' types and vocabulary to an ACL package in the infrastructure layer, and pass only your own models to the domain layer.
If you bring the other side's response structure straight into the domain, domain logic and tests break on every spec change.
Build the ACL from three elements: Facade, Translator, and Adapter.
The Facade is the entry point that hides HTTP or gRPC communication details (endpoints, auth headers, status checks, decoding).
The Translator translates external vocabulary (authorized / captured) into domain vocabulary (pending / completed) and also absorbs differences in datetime formats.
Always return an error for unknown values so that changes on the other side are detected early.
The Adapter implements the interface the UseCase layer defined on the consumer side, and does only the orchestration of translate, communicate, translate.
The Interactor does not know the ACL exists and is written purely in terms of domain models. Even if REST changes to gRPC, only the Facade and the external types are swapped; the three-element skeleton stays the same.

## When it applies, and exceptions
- Adopt when: the external API's model differs substantially from your domain, specs change frequently, or you integrate with a legacy system
- Do not adopt when: it is another service from the same team with a shared domain language. A translation layer tends to be wasted
- Do not adopt when: it is a standard library such as a DB or cache. The model divergence is small
- Putting an ACL on every integration makes maintaining translation code a burden. If the external model is close to the domain, a thin implementation that omits the Facade or Translator is fine
- Make the Translator a unit-test target. Being able to verify state mapping, unknown-value errors, and request conversion without calling the external system is the advantage of an ACL

## How to spot violations
- External response structs with JSON tags or Protocol Buffers generated types are imported from the domain layer or UseCase layer
- The domain or Interactor branches on external API state strings such as `"captured"`
- HTTP client operations or JSON decoding are written directly inside the Adapter, mixed with the Facade
- Unknown state values from outside are ignored and dropped to a default, so spec changes go unnoticed
- The interface for the external integration is defined on the infrastructure layer side, and the UseCase layer depends on it

## Source
- Ch. 12 「腐敗防止層（ACL）〜外部 API の形式を翻訳する〜」 (Anti-corruption layer (ACL): translating external API formats) https://github.com/135yshr/documents/blob/main/books/go-service-design/anti-corruption-layer.md
