# Ch. 15: Where to open transactions

## The book's default
When you find yourself wanting a transaction, first question the aggregate boundary; if you still need to update multiple aggregates synchronously, hand only the "authority to decide the boundary" to the UseCase layer via a TxManager.
- One transaction modifies at most one aggregate. Multiple tables that are always updated together are a single aggregate, and if the transaction is closed inside the Repository's `Save`, no control appears in the UseCase layer.
- Consistency across aggregates is, in most cases, adequately handled by eventual consistency through domain events.
- For the remaining cases where "multiple aggregates must be updated immediately as one unit of work", define a TxManager interface of the form `Do(ctx, fn func(ctx) error) error` in the UseCase layer. The Interactor just calls multiple Repositories inside `Do`; `*sql.Tx` never appears. If fn returns an error, roll back; if nil, commit.
- The implementation lives in the infrastructure layer. It puts the started Tx on the context and passes it to fn; a Repository uses the Tx if one is on the context, and `*sql.DB` otherwise (received through an unexported interface bundling the methods common to both).
- Having the UseCase layer hold `*sql.DB` and call `BeginTx` violates the dependency rule, and also means tests require a DB.

## When it applies, and exceptions
- The update fits within one aggregate → open the transaction inside the Repository. Do not expose it to the UseCase layer.
- Multiple aggregates updated synchronously → TxManager. The boundary is in the UseCase layer; begin and commit are in the infrastructure layer.
- Spans aggregates but immediate consistency is not required → domain events + eventual consistency.
- Event publication must not be lost → Transactional Outbox: write to an outbox table in the same transaction, and have a separate process deliver it. Simply calling `outbox.Save` inside `Do` gets it on board.
- Putting the Tx on the context is acceptable only under two conditions: the key type is an unexported type in the infrastructure layer's postgres package, and things work correctly by falling back to `*sql.DB` when no Tx is present. If you dislike hidden hand-offs, you can instead add an explicit queryer argument to Repository methods, but then an infrastructure-driven argument leaks into the domain layer's interface. If you prioritize interface purity, use the context approach.
- Unit of Work (tracking changes and committing them all at the end) is a tool from language ecosystems whose ORMs have change tracking. Implementing it yourself in Go is expensive; treat the TxManager as a simplified version that achieves only that purpose without change tracking.

## How to spot violations
- The UseCase layer imports `database/sql`, or an Interactor struct has `*sql.DB` or `*sql.Tx` fields.
- A Repository interface method has an argument like `tx *sql.Tx` (an infrastructure type leaking into the domain layer).
- The context key type is exported, and the UseCase layer or interface layer pulls the Tx out with `ctx.Value`.
- Two tables that are almost always updated together are split into separate Repositories, and a transaction is opened in the UseCase layer to cope with that (the aggregate boundary is suspect).
- Events are published after commit, with no countermeasure for publication failure.

## Source
- Ch. 15「トランザクション管理〜UseCase 層で境界を引く〜」(Transaction management: drawing the boundary in the UseCase layer) https://github.com/135yshr/documents/blob/main/books/go-service-design/transaction-management.md
