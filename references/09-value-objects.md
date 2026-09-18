# Ch. 9: How far to go with value objects

## The book's default
Create value objects only for values that meet at least one of four conditions; leave everything else as primitives.
The conditions are (1) there is format or range validation, (2) the same primitive type is used with multiple meanings and could be mixed up, (3) there is domain-specific behavior such as forbidding addition across currencies, (4) input normalization is needed. Meeting even one makes it worth creating.
Use two representations. A Named Type (`type UserID string`) is lightweight and gives only type safety, suited to IDs and status constants. A struct + unexported fields + New function comes with validation, suited to emails, money, and phone numbers.
When in doubt, decide by "does this type prevent bugs, or express domain knowledge?" If both are No, a primitive is the Go way.

## When it applies, and exceptions
- Turning everything into value objects piles up New, getter, and `String()` boilerplate, makes conversion to DTOs and DB records verbose, and leaves new members wondering "how is this different from string?" Do not create them where the benefit does not justify the cost
- A name with nothing more than a "not empty" check, a free-text bio, and a createdAt that `time.Time` covers stay primitives. Validating inside the entity's New function is enough
- A Named Type can be assigned without validation (an empty string passes). Add a `NewXxx` if needed. Start enums at `iota + 1` so the zero value can be detected as unset, but out-of-range values can only be blocked by a constructor
- A struct value object can be compared with `==` if it has only comparable fields. If it holds a map or slice, implement `Equals`
- Do not forget the risks on the Primitive Obsession side either: argument mix-ups are not caught at compile time, format checks get scattered, and the type does not convey the domain meaning

## How to spot violations
- Getter chains like `user.Name().Value()` line up in conversions such as `toResponse` (signal of excess)
- There is a wrapper type with no rules or methods that only returns `String()`
- A function takes three or more arguments of the same primitive type in a row, and swapping their order still compiles (signal of deficiency)
- Email address or phone number format checks are duplicated across the interface layer, Interactor, and model
- Value object fields are exported and instances can be created without going through the constructor

## Source
- Ch. 9 「値オブジェクトはどこまで作るか〜4つの判断基準〜」 (How far to go with value objects: four criteria) https://github.com/135yshr/documents/blob/main/books/go-service-design/value-objects.md
