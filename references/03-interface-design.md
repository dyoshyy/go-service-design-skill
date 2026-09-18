# Ch. 3: Where do interfaces go?

## The book's default
Define interfaces in the consuming package, with only the methods the consumer needs.
By Go convention, an interface belongs to "the side that uses the value". `io.Reader` is defined by the reader; `os.File` does not know it exists.
Thanks to implicit interfaces, the providing side (an Interactor or a Repository implementation) can simply export a struct, and the consuming side (a Handler in the interface layer, or another Interactor) keeps a private interface of the one or two methods it wants; that alone achieves dependency inversion.
The smaller the interface, the stronger it is as an abstraction. Most standard-library interfaces have one or two methods; by that standard a Repository with five to eight methods is fat.
Carving out an interface with a single implementation "for the future" is an anti-pattern known as a speculative interface. The rule of thumb is accept interfaces, return structs.
The problem is never the number of interfaces; it is fat interfaces and interfaces in the wrong place.

## When it applies, and exceptions
- Export the Interactor as a struct and do not create an Input Port (an interface in which the UseCase layer defines how it is to be called). The calling Handler makes an interface of only the methods it needs
- The same applies when an Interactor uses another Interactor: the consuming side defines a private interface
- An Output Port, the abstraction over an external service (LLM, message queue, auth provider), stays a shared interface in the UseCase layer only when it is shared by multiple Interactors and there is a track record or a plan of swapping implementations. If only one Interactor uses it, define it on the consuming side
- If there is neither a track record nor a plan of swapping implementations, don't create the interface at all
- When a Repository interface grows more methods, split it into Reader / Writer. A read-only Interactor depends on the Reader alone, and a single struct in the infrastructure layer implicitly satisfies both
- For a small CRUD-centric app, Repository interfaces plus struct UseCases are enough. The separation pays off in projects with many external integrations or multiple switchable strategies
- Whether DI is manual or via a DI container, pass or register the struct directly. Since the interface lives on the consuming side, it is satisfied automatically

## How to spot violations
- There is a directory like `usecase/port/input/`, and Handlers reference interfaces defined there
- An exported interface has exactly one implementation, with no implementation other than test mocks
- A Repository interface exceeds five methods, or a read-only Interactor depends on an interface that includes Save or Delete
- Registrations in the DI container are as interfaces, in a form like `dig.As(new(SomeInterface))`
- The interface is defined in a different package from where it is used, and you can't tell "where is this used" without searching
- There is no `var _ Interface = (*Impl)(nil)`, so a missed implementation after an interface change isn't caught at compile time

## Source
- Ch. 3 「Go の interface 設計〜利用側で定義し、小さく保つ〜」 (Designing Go interfaces: define on the consuming side, keep them small) https://github.com/135yshr/documents/blob/main/books/go-service-design/interface-design.md
