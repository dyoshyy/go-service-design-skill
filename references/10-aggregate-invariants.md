# Ch. 10: How to protect invariants

## The book's default
An aggregate with invariants should keep its fields unexported, restrict construction to a New function, and restrict changes to behavior methods.
Go has no access modifiers, but lowercase-initial unexporting works at the package level, which is enough encapsulation.
Exporting fields scatters the responsibility for keeping the state valid across callers, and the aggregate becomes something that "can only hope".
Validate in the New function so that an instance in an invalid state can never be created, and copy any slice you receive to take ownership.
Derived values such as totals are computed internally rather than passed in from outside, and state-transition checks are also enclosed in methods.
Keep getters to the ones actually needed for persistence or response assembly; do not inspect state and branch on it outside.
For restoring from the infrastructure layer, provide a separate Reconstruct function that skips validation, and constrain its use by naming and review.
Concurrency control is not the aggregate's job; handle it with locks or transactions on the repository side.

## When it applies, and exceptions
- Adopt when: the struct has consistency rules across fields, such as "at least one line item" or "no additions after confirmation"
- Do not adopt when: the struct is a container for settings or data whose fields may be read and written independently (treat it like Response or Config in the standard library)
- Unexporting is a package boundary, so code in the same package can still touch the fields. Consider splitting the model package if it grows too large
- The problem that a zero value can be created is, in practice, adequately handled by detecting an empty id and the like inside methods and rejecting it
- For extracting values for persistence, it is fine to start with exported getters. Once logic leakage into the interface layer becomes noticeable, move to the Snapshot approach (return the state all at once as a flat exported struct)
- Do not let DB columns or JSON dictate the shape of the Snapshot. Mapping to column names or JSON tags is done in the infrastructure layer

## How to spot violations
- Comparisons like `order.Status() == ...` are written in the interface layer or an Interactor, deciding whether a transition is allowed outside the aggregate
- `Order{}` or struct literals are assembled directly outside the model package
- `Reconstruct*` has callers other than repository implementations
- The New function stores an argument slice as-is, or receives a total as an argument
- Every field has a getter, and `Items()` is exported for list display (a sign that a Read Model should be considered)

## Source
- Ch. 10 「集約の不変条件〜非公開フィールドで守る〜」 (Aggregate invariants: protecting them with unexported fields) https://github.com/135yshr/documents/blob/main/books/go-service-design/aggregate-invariants.md
