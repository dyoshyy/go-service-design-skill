# Ch. 16: What to test, and how far

## The book's default
Vary the test level by layer, and make domain-layer unit tests the main battleground.
- The domain layer (value objects, aggregates, domain services) has no external dependencies, so you can write fast, stable unit tests. It has the highest return on investment.
- The UseCase layer gets component tests that replace Repositories and event buses with test doubles and verify the Interactor's flow. Real domain objects are used, and there is no dependency on infrastructure-layer implementations.
- The infrastructure layer gets integration tests against a real DB. Set up the connection in `TestMain`, and register `t.Cleanup` before calling Save. Spinning up a temporary PostgreSQL with testcontainers-go is another option.
- E2E tests of the interface layer (HTTP handlers, etc.) are heavily framework-dependent and outside the scope of this chapter.
- For value objects, test the constraints at construction time and equality; for aggregates, test not internal fields but "the changes observable from outside after executing a command". If state transitions are complex, cover them with table-driven tests and cross-check against the state transition diagram to find gaps.
- Test doubles are hand-written stubs, spies and fakes placed in the test file. Go interfaces are satisfied implicitly, and for small interfaces the overhead of a mock generation tool outweighs its benefit.

## When it applies, and exceptions
- In domain service tests, the only things replaced are the Repository and external-service ports. Do not mock value objects or entities.
- Mocks that verify call counts (gomock, etc.) are a dependency on implementation details; heavy use lowers refactoring resilience. If stubs, spies and fakes suffice, use those.
- When an interface has many methods, or call verification is truly needed, a mock library is also reasonable.
- Separate tests that need a DB with a build tag such as `//go:build integration`, and run them only in CI. Run the domain layer frequently and locally with `go test ./domain/...`.
- If a test feels hard to write, question the aggregate boundary or the interface design rather than the test technique.

## How to spot violations
- Aggregate tests verify internal state directly through unexported fields or setters.
- Domain-layer tests require a DB or external API, or mock the domain objects themselves.
- UseCase-layer tests import the Repository implementation from the postgres package.
- Generated mocks are used for a one- or two-method interface, with rows of call-count assertions like `Times(1)`.
- Test names do not follow `Test{FunctionName}_{Scenario}`, and Arrange / Act / Assert are not separated by blank lines. Helpers lack `t.Helper()`.

## Source
- Ch. 16「テスト戦略〜どの層を何でテストするか〜」(Testing strategy: which layer to test with what) https://github.com/135yshr/documents/blob/main/books/go-service-design/testing-strategy.md
