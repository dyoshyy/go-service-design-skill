# Ch. 6: How far to strip out DB dependencies

## The book's default
Design at Level 1 from the start (Repository interface in the domain layer, implementation in the infrastructure layer).
The Repository's job is not to hide SQL; it is to express set operations on domain objects in the domain's vocabulary. Cutting an interface costs little, and you get testability, clear dependency direction, and resilience to future change from day one. Thanks to Go's implicit interfaces, raising the level later leaves the implementation code unchanged (only wiring and tests need fixing).

| Level | What it is | Where it fits |
|---|---|---|
| 1 Full abstraction | interface in the domain layer, implementation in the infrastructure layer | DB may change, multiple storages, model and schema diverge |
| 2 Thin abstraction | Use sqlc / ent generated types and interfaces directly in the UseCase layer | Fixed DB, schema ≒ model, medium scale |
| 3 Direct dependency | Interactor holds `*sql.DB` etc. directly | Prototypes, small scale, only a handful of DB access points |

## When it applies, and exceptions
- Conditions for Level 1: the DB type may change, multiple storages are combined (e.g. RDB + Redis + S3), the domain model and DB schema differ significantly, several people touch the same Repository, or you want to swap the DB out in unit tests
- Conditions for relaxing to Level 2: the DB is fixed, schema and model nearly match, and something like sqlc's `Querier` (`emit_interface: true`) can serve as the mock as-is. Note that the UseCase layer then depends on infrastructure-derived types, which strictly violates the dependency rule, and errors like `sql.ErrNoRows` leak into the UseCase layer (the translation from Ch. 7 is needed)
- Conditions for relaxing to Level 3: a prototype or small tool where tests run against a real DB via testcontainers etc. Raise to Level 1 when the need arises
- Per-tool guidance: database/sql and GORM are Level 1 (SQL and scanning bleed into the UseCase layer; `*gorm.DB` is hard to mock), ent is 1-2 (in-memory possible with enttest), sqlc is 2-1 (narrowing to only the methods the caller needs gives an ISP-compliant Level 1)
- Cutting an interface is justified when the goal is faster unit tests or a CI that needs no DB. If integration tests are enough, tests can be written without an interface

## How to spot violations
- The Interactor's struct fields hold concrete types such as `*sql.DB`, `*gorm.DB`, or a generated `Queries` struct
- SQL strings or `Scan` handling appear in UseCase-layer files
- The Interactor references an infrastructure-layer concrete type directly, like `postgres.TaskRepository`
- Tests assert on SQL string matches (as with go-sqlmock) and break easily under refactoring
- Repository interface method names use DB vocabulary like `Query` `Exec` rather than domain vocabulary like `FindByID` `Save`

## Source
- Ch. 6 「Repository の抽象化レベル〜DB 依存をどこまで剥がすか〜」 (Repository abstraction levels: how far to strip out DB dependencies) https://github.com/135yshr/documents/blob/main/books/go-service-design/repository-levels.md
