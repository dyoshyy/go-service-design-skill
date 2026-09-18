# Ch. 11: Reducing coupling between aggregates

## The book's default
Limit each transaction to updating one aggregate, and propagate effects to other aggregates through domain events.
When an Interactor updates order, stock, and points in sequence, tight coupling, lock contention, modifications on every new requirement, and mock hell all happen at once.
The aggregate accumulates events internally when its state changes; the Interactor publishes them together after Save and then clears the published ones.
Name events in the past tense (OrderConfirmed) to express "a fact that happened", and give them enough information that receivers need not re-read the source.
Define the EventPublisher used for publishing as a port in the UseCase layer, and put the channel implementation or NATS implementation in the infrastructure layer.
The port has only Publish. Subscription registration is wiring work at startup, so it does not go into the Interactor's dependencies.
Stock deduction and point granting are each registered at startup as independent handlers, and the publisher does not know they exist.
This makes consistency between aggregates eventual. If immediate consistency is needed, question how the aggregate boundaries were drawn.

## When it applies, and exceptions
- Adopt when: you want to update another aggregate in response to an operation on one aggregate. Valid within the same BC as well
- An in-process bus built on channels has the constraints of at-most-once delivery, loss on restart, and serial handler execution. Limit it to starting small in a single process
- Atomicity of Save and Publish is not guaranteed. If reliability is needed, use the Outbox pattern (write to an event table in the same transaction and have a separate worker deliver) or an external MQ
- For inter-service delivery, start with NATS (lightweight, at-least-once with JetStream) and consider RabbitMQ when routing becomes complex
- Notifications across BCs should not pass domain events through as-is; convert them into integration events tailored to the external concern
- Assume duplicate delivery. Make handlers idempotent by event_id, and guarantee certainty with DB unique constraints (application-side checks are merely an optimization)

## How to spot violations
- A single Execute repeats Find, modify, and Save against multiple repositories
- Event names are imperative, like ConfirmOrder
- A handler re-fetches the source aggregate right after receiving the event (the event lacks information)
- The domain layer imports Bus or messaging-infrastructure types
- The event type passed to Subscribe is a raw string rather than a constant. If it drifts, events silently disappear

## Source
- Ch. 11 「ドメインイベント〜集約間の結合を減らす〜」 (Domain events: reducing coupling between aggregates) https://github.com/135yshr/documents/blob/main/books/go-service-design/domain-events.md
