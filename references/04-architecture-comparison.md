# Ch. 4: Which architecture should you adopt?

## The book's default
Don't choose by style name. Start from the smallest split that protects the direction of dependencies.
Hexagonal (Ports & Adapters), Onion, and Clean Architecture are the same thing in that dependencies point toward the center; the differences are only vocabulary and emphasis.
Hexagonal prescribes no number of layers, just "center and outside", and treats the driving side (HTTP, CLI) and the driven side (DB, external APIs) symmetrically. Onion calls the center the Domain Model, in DDD terms. Clean makes Use Cases an explicit, separate circle and attempts to unify its predecessors.
Only Vertical Slice is on a different axis: it cuts by feature (vertical) rather than by technical layer (horizontal). It is not mutually exclusive with layering; cut the outside into modules and the inside of each module into layers, and both can be used together.
Go's implicit interfaces remove the need to declare port implementations, which makes them a good fit for a two-layer core-and-adapter structure (hexagonal as such).
As long as the direction of dependencies is protected, changing the structure later is not hard.

## When it applies, and exceptions
- Small tools, single-feature microservices → two layers, core and adapter (hexagonal-style)
- Services with multiple features → module split × three to four layers inside each module (Vertical Slice × Clean)
- Large variance in complexity between features → prioritize the module split and vary layer thickness per module. Slices that are CRUD-only get fewer layers; only logic-heavy slices get thick ones
- When a service that started with two layers grows, split core into a domain layer and a UseCase layer. There is no need to start with four layers
- When in doubt, pick the smaller option. Waiting until a reason to add a layer appears is soon enough
- Whatever the structure, the one thing to protect is the direction of dependencies. Whether a UseCase layer is needed is decided in Ch. 5; the unit of module split in Ch. 13

## How to spot violations
- Design time is being spent on a debate about names, "hexagonal or clean"
- Every feature is forced into the same four layers, so even CRUD-only modules have a near-empty UseCase layer or domain service
- The split is by layer first, so one feature change crosses four directories: entities / usecases / adapters / frameworks
- A small single-feature service has both four layers and a module split laid out from the start
- "Which way do dependencies point" is not on the checklist in structure discussions

## Source
- Ch. 4 「アーキテクチャスタイルの比較〜ヘキサゴナル・オニオン・Vertical Slice〜」 (Comparing architecture styles: Hexagonal, Onion, Vertical Slice) https://github.com/135yshr/documents/blob/main/books/go-service-design/architecture-comparison.md
