# Ch. 2: How many layers do you need?

## The book's default
Don't count layers; protect the direction of dependencies.
The only thing the concentric-circle diagram prescribes is that "dependency arrows always point inward (toward the domain layer)". It says nothing about the number of layers, directory layout, naming, or how thick each layer is.
Two layers (core and adapter) are still clean architecture as long as the inside knows nothing about the outside; conversely, five layers mean nothing if the domain layer imports the infrastructure layer.
DIP is what upholds this direction: have the UseCase layer define abstractions and the infrastructure layer implement them, and swapping the DB or an external API never ripples inward.
In Go, implicit interfaces let the implementing side satisfy an interface without knowing it exists, so this inversion is natural to write.
Outer layers are not "allowed to be dirty"; they merely "don't depend on business logic". Separation of responsibilities and testability are required inside and outside alike.

## When it applies, and exceptions
- A small tool or single-feature microservice is fine with two layers: a core holding business logic and interface definitions, and an adapter holding HTTP, DB, and external APIs
- A service with multiple features should be split by module (feature) first, then into domain / UseCase / interface / infrastructure within each. Splitting by layer first scatters one feature's files across four directories
- Define interfaces on the consuming side (the UseCase layer or another inner layer), with only the methods that consumer needs. The implementing Repository may have more methods
- Put dependency injection and the compile-time check `var _ Interface = (*Impl)(nil)` in the composition root (main or DI setup), which already knows every dependency. Writing them in the infrastructure layer adds dependencies other than outer-to-inner
- It is acceptable for implementing-side methods to take inner types (domain Entities, etc.) as arguments or return values. The problem is never that a dependency exists, only its direction
- Add a layer only once the reason to add it has actually appeared

## How to spot violations
- A domain-layer file imports the infrastructure layer (cache, postgres, etc.)
- UseCase-layer code references `*sql.DB`, `*http.Request`, or driver-specific types directly
- An interface is defined in the implementing package (infrastructure layer) and the UseCase layer imports it
- Dependencies are assembled not in main or DI setup but by `new`-ing concrete types inside each layer
- In the thought experiment of adding a new outer layer (another DB, another input path), existing inner layers would need changes
- Dependency direction is not mechanically checked by a linter such as `depguard` or `go-cleanarch`

## Source
- Ch. 2 「依存性ルール〜同心円図の正しい読み方〜」 (The dependency rule: reading the concentric-circle diagram correctly) https://github.com/135yshr/documents/blob/main/books/go-service-design/dependency-rule.md
