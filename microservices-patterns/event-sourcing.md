## Chapter 6: Developing Business Logic with Event Sourcing
Chapter 6 introduces **Event Sourcing**, a powerful alternative to traditional state-based persistence. Instead of storing the current state of an object (like a row in a table), you store a sequence of state-changing **events**.
### 6.1 The Problem with State-Based Mapping
In traditional persistence, you map objects to tables (ORM). This has several drawbacks:
 * **Object-Relational Mismatch:** Complex object graphs are hard to map to flat tables.
 * **Lack of Audit History:** You only know the *current* state. Finding out *how* the system reached that state requires separate, often incomplete, audit logs.
 * **Concurrency Issues:** Frequent updates to the same record can lead to lock contention.
### 6.2 The Event Sourcing Pattern
Event Sourcing persists an aggregate by saving its history of events in an **Event Store**. To get the current state, you "replay" these events from the beginning.
 * **The Event Store:** A hybrid of a database and a message broker. It provides an API for inserting events and subscribing to them.
 * **Snapshots:** To avoid replaying thousands of events every time, the system periodically saves a "snapshot" of the state.
 * **Evolution:** Since events are immutable, you handle schema changes by supporting multiple event versions.
## Technical Examples: Event Sourcing Implementation
### C# Example: Event-Sourced Aggregate
In C#, you can implement a base class for your aggregates that tracks uncommitted events.
```csharp
public abstract class AggregateRoot
{
    private readonly List<object> _changes = new();
    public int Version { get; protected set; } = -1;

    public IEnumerable<object> GetUncommittedChanges() => _changes;

    protected void ApplyChange(object @event)
    {
        Apply(@event);
        _changes.Add(@event);
    }

    protected abstract void Apply(object @event);
}

public class Account : AggregateRoot
{
    public decimal Balance { get; private set; }

    public void Deposit(decimal amount)
    {
        ApplyChange(new MoneyDeposited(Guid.NewGuid(), amount));
    }

    protected override void Apply(object @event)
    {
        if (@event is MoneyDeposited e)
        {
            Balance += e.Amount;
        }
    }
}

```
### Python Example: Event Replay Logic
Python’s dynamic nature makes it easy to map event types to handler methods during a replay.
```python
class OrderAggregate:
    def __init__(self):
        self.order_id = None
        self.status = "NONE"
        self.items = []

    def apply(self, event):
        """Standard dispatcher for event types"""
        event_type = event['type']
        if event_type == "ORDER_CREATED":
            self.order_id = event['id']
            self.status = "CREATED"
        elif event_type == "ITEM_ADDED":
            self.items.append(event['item_id'])

    @classmethod
    def rebuild_from_history(cls, events):
        instance = cls()
        for event in events:
            instance.apply(event)
        return instance

# Usage
history = [
    {"type": "ORDER_CREATED", "id": "ORD-123"},
    {"type": "ITEM_ADDED", "item_id": "SKU-99"}
]
order = OrderAggregate.rebuild_from_history(history)

```
## Key Takeaways
 1. **100% Accurate Audit Log:** The history is the "Source of Truth," making debugging and regulatory compliance much easier.
 2. **Temporal Querying:** You can reconstruct the state of the system at any specific point in time.
 3. **Complexity Trade-off:** Event Sourcing has a steeper learning curve and requires a shift in how you think about data—you can't just run a SELECT * query (this leads into Chapter 7).
