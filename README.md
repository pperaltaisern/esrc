# esrc
A Go library for building event sourced applications.


## Features
- Provides an event store interface to store and retrieve Aggregates (event streams), Events and Snapshots
- Ensures optimistic concurrency control.
- [Relays](./relay) events with guaranteed order.
- Provides a repository struct for aggregates, an abstraction layer on top of the event store that persists and rehydrates aggregates.
- Provides a mechanism for your aggregates to raise events and track changes.

## Event Store implementations

- PostgreSQL (https://github.com/pperaltaisern/esrcpg)

Implementations must pass the acceptance test suite in [./esrctesting/event_store.go](./esrctesting/event_store.go)

## CQRS
Supporting CQRS is not the goal of this library, but it supports relaying of the events with guaranteed order. Adapting CQRS focused libraries to the relay package enables event sourcing + CQRS.

### Adapters
- Watermill (https://github.com/pperaltaisern/esrcwatermill)

## Examples

- go-example-financing (EventSourcing + CQRS) (https://github.com/pperaltaisern/go-example-financing)

## API Overview

**EventStore** is the library's core, understands only of aggregate's type and ID, and it's events and snapshots.

![EventStore operations sample](/doc/event_store.jpg "EventStore operations sample")

On top of that there is the **Repository**, that encapsulates the **EventStore** and interacts with **Aggregates**.


```go 
type Repository[T Aggregate] struct {...}

func (r *Repository[T]) FindByID(ctx context.Context, id ID) (T, error) { ... }
func (r *Repository[T]) Contains(ctx context.Context, id ID) (bool, error) { ... }
func (r *Repository[T]) Update(ctx context.Context, aggregate T) error { ... }
func (r *Repository[T]) Add(ctx context.Context, aggregate T) error { ... }
func (r *Repository[T]) UpdateByID(ctx context.Context, id ID, update func(aggregate T) error) error { ... }
```

The **Aggregate** is the interface that domain aggregates must satisfy in order to use the **Repository**.

```go
type Aggregate interface {
	ID() ID
	InitialVersion() int
	Changes() []Event
	Snapshot() ([]byte, error)
}
```

Optionally, the **EventRaiserAggregate** can be embedded in domain aggregates to raise and keep track of events:

```go
type MyAggregate struct {
    esrc.EventRaiserAggregate
    ...
}

func NewMyAggregate(id uuid.UUID) *MyAggregate {
	agg := &MyAggregate{}
	agg.EventRaiserAggregate = esrc.NewEventRaiserAggregate(agg.onEvent)

	e := NewMyAggregateCreatedEvent(id)
	agg.Raise(e)
	return inv
}

func (agg *MyAggregate) onEvent(event esrc.Event) {
	switch e := event.(type) {
	case *MyAggregateCreatedEvent:
		agg.id = e.ID
	}
}

```