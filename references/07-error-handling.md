# Ch. 7: How to classify errors

## The book's default
Translate errors at each layer boundary, and fix where the translation happens.
The domain layer returns only sentinel errors from `errors.New` (it knows nothing of HTTP statuses or codes). The infrastructure layer translates DB-specific errors into domain sentinels (no record → `ErrXxxNotFound`, unique constraint violation → `ErrDuplicateXxx`) and wraps everything else with `%w` to propagate. The UseCase layer (Interactor) converts sentinels into a CodedError carrying a code and HTTP status. The interface layer has exactly one aggregate handler that extracts the CodedError with `errors.As`; anything that is not a CodedError gets logged and returns a generic 500.
Three categories: domain errors (rule violations, 400/409/422), application errors (not found / no permission, 404/403), infrastructure errors (connection failures / timeouts, 500/502/503). More than the classification, what pays off is making explicit "where the conversion happens".

## When it applies, and exceptions
- It is fine to start with sentinels only. Introduce CodedError once `errors.Is` branches start multiplying in the interface layer. The migration can be gradual
- Repository implementations must not return CodedError. The infrastructure layer would then know a type that embeds HTTP statuses, breaking the responsibility split. Domain sentinels are as far as it may go
- The infrastructure layer depending on domain sentinels is the correct dependency direction, so it is allowed. Testing that translation logic, however, needs a DB mock or testcontainers
- When in doubt, split on "fact vs. judgment". Translating the fact that a record was missing or a key was duplicated is the Repository's job; promoting it to an application-level meaning like 404 or 409 is the UseCase layer's job
- For field-level validation errors, add details to CodedError or create a dedicated type (RuleViolation in Ch. 8)
- Security: never return infrastructure error messages or stack traces to the client. For resources whose existence should be hidden, fold 403 into 404. Do not let login failure reasons reveal whether a user exists

## How to spot violations
- The interface-layer handler has `errors.Is(err, model.ErrXxx)` stacked vertically, and every new sentinel adds another if
- An infrastructure-layer package imports `apperror` or `net/http`'s StatusXxx
- The Interactor compares `gorm.ErrRecordNotFound`, `sql.ErrNoRows`, or pg error codes directly
- On unexpected errors, `err.Error()` is written straight into the response
- `fmt.Errorf` uses `%v`, so `errors.Is` / `errors.As` cannot walk the chain

## Source
- Ch. 7 「エラーハンドリング設計〜ドメインエラーとインフラエラーの分離〜」 (Error handling design: separating domain errors from infrastructure errors) https://github.com/135yshr/documents/blob/main/books/go-service-design/error-handling.md
