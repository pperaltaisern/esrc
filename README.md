# esrc
A Go library for building event sourced applications.


## Features
- Provides an event store interface to store and retrieve Aggregates (event streams), Events and Snapshots
- Ensures optimistic concurrency control.
- [Relays](./relay) events with guaranteed order.
- Provides a repository struct for aggregates, an abstraction layer on top of the event store that persists and rehydrates aggregates.
- Provides a mechanism for your aggregates to raise events and track changes.


## API Overview

![EventStore operations sample](/doc/event_store.jpg "EventStore operations sample")

- **Event Store**: Is the interface for persisting and retrieve Aggregates as event streams, with their Events and Snapshots.


- **Aggregate**: Is the interface that any [Aggregate](https://martinfowler.com/bliki/DDD_Aggregate.html) has to implement in order to use a Repository.
- **Repository**: Is a layer built on top of the Event Store which translates between Aggregate and the Event Store. 
