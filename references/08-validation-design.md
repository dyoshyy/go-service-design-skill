# Ch. 8: Where to write validation

## The book's default
Split validation across three layers (interface, UseCase, domain) and fix what each layer protects.
The interface layer looks only at input shape (whether the JSON parses, required fields, string/array length, UUID or email format). Write it declaratively with struct tags or middleware, and bring in no domain knowledge. The UseCase layer checks consistency that requires querying the Repository (existence of referenced records, uniqueness, permissions, conditions spanning multiple aggregates). The domain layer enforces invariants in value object and entity constructors, so that invalid values cannot exist no matter the entry point (always-valid).
Domain-layer errors express "which rule was violated" and carry no field names. The mapping from rule name to field name lives in the interface layer. Multiple violations are returned together.

## When it applies, and exceptions
- The reason for three layers is that REST is not the only entry point. When gRPC, CLI, or batch jobs are added, relying on interface-layer checks means reimplementing them
- Upper bounds appearing in both the interface layer and the domain layer is intentional. The former is early feedback; the latter is the route-independent last line of defense. Their roles differ
- Existence and duplicate checks are kept out of the domain layer because an aggregate depending on the Repository blurs the boundary. When the same rule is needed by multiple use cases, move it to a domain service
- The UseCase-layer uniqueness check is for early feedback. The last line of defense against concurrent requests is the DB unique constraint; on violation the Repository returns a domain sentinel (Ch. 7)
- Entity constructors take primitives and build value objects internally. Re-validating already-validated values is wasteful, but not shifting the responsibility for missed validation onto the caller takes priority
- SQL injection and XSS cannot be prevented by input validation. Prepared statements and output escaping are the primary defenses; validation is supplementary

## How to spot violations
- The same check is duplicated across the interface layer, Interactor, and model, and nobody can say which one is authoritative
- Value object fields are exported, or the constructor does not return error and invalid values can be built
- A domain-layer constructor contains Repository calls or JSON format checks
- A domain error type has HTTP-request-derived concepts like `Field`
- State changes are direct assignments like `task.status = Done`, with no table of allowed transitions

## Source
- Ch. 8 「入力バリデーション設計〜3層での役割分担〜」 (Input validation design: dividing roles across three layers) https://github.com/135yshr/documents/blob/main/books/go-service-design/validation-design.md
