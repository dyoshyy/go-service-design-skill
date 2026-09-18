# Ch. 17: How to keep the rules enforced

## The book's default
Do not rely on visual review for the dependency rule; check imports with static analysis and block violations with a required CI check.
- The larger the team, the more accidents like the domain layer importing the infrastructure layer occur, and if review misses them, the decay proceeds quietly.
- The allowed dependency directions are interface layer → UseCase layer → domain layer, and infrastructure layer → UseCase layer / domain layer. The inner must not know the outer.
- The mainstay is depguard (built into golangci-lint). In the `rules` of `.golangci.yml`, write the layer path globs in `files`, and the forbidden pkgs plus a `desc` in `deny`. The reason appears in the error, so developers can fix it on their own.
- Complement it with kcmvp/archunit. Declare layers with `ArchLayer` and verify rules inside `go test`, e.g. `Layers("Domain").ShouldNotRefer(Layers("UseCase","Interface","Infrastructure"))`.
- To survey the current state, use go-cleanarch. It infers layers from directory names (domain / usecase / interface / infrastructure), so no configuration is needed. If you want YAML-defined rules on the CLI, use arch-go.
- Run it on every PR with GitHub Actions and register it as a required status check under branch protection. The essence is "make it a required check", more than the choice of tool.

## When it applies, and exceptions
- If you already use golangci-lint, make depguard the mainstay. If you want a quick look at the current state, go-cleanarch. For complex rules, the programmable archunit.
- Introduce it into existing projects in stages: go-cleanarch for the overall picture → depguard in warning-only mode to visualize → fix violations module by module → promote to a required check at zero violations.
- `_test.go` files may import test mocks and fixtures, so exclude them by adding `!**/*_test.go` to depguard's `files`.
- Restrict only the direction between layers. Leave imports within the same layer free, and do not make the rules too strict.
- In `desc`, write "why it is forbidden" and "what to do instead" (e.g. define an interface in the domain layer and implement it in the infrastructure layer).
- `interface` is a Go reserved word, so it cannot be a package name. Use a subpackage such as `interface/rest`, or use `presentation/` or `adapter/`.

## How to spot violations
- A domain-layer file imports `infrastructure/postgres` or similar, and a domain service directly news up a Repository implementation.
- The UseCase layer takes a Request struct from `interface/rest/dto` as the argument of Execute (making gRPC or CLI support difficult).
- The dependency check is left to manual local runs and is not in CI, or is in CI but not a required check.
- The depguard rules forbid uniformly across all packages, rejecting even test files and imports within the same layer.
- The violation message says only "forbidden" and does not show how to fix it.

## Source
- Ch. 17「依存性ルールを CI で守る〜静的解析による自動チェック〜」(Enforcing the dependency rule in CI: automated checks via static analysis) https://github.com/135yshr/documents/blob/main/books/go-service-design/enforce-dependency-rule.md
