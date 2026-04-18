My apologies for jumping ahead—I got a bit carried away! Let's get back on track.
Since you asked **when** Event Sourcing is useful, here is a "Markdown copy-paste ready" recreation of Chapter 7. This chapter addresses the specific problem created by the patterns in Chapter 6: if you only store a list of events, how on earth do you query them efficiently?
## Chapter 7: Implementing Queries in a Microservice Architecture
In a microservice architecture, data is private to each service. This creates a "Distributed Data" problem: you can no longer perform a simple SQL JOIN to get a view that combines data from multiple services (like showing an Order alongside Customer and Shipping details).
### 7.1 The API Composition Pattern
The simplest way to query across services. An "API Composer" (which could be a frontend or a dedicated service) calls multiple services and joins the results in memory.
 * **Pros:** Simple to implement.
 * **Cons:** Increased overhead from multiple network calls; reduced availability (if one service is down, the whole query might fail).
### 7.2 The CQRS Pattern (Command Query Responsibility Segregation)
For complex queries, Richardson advocates for **CQRS**. This pattern separates the application into two distinct parts:
 1. **The Command Side:** Handles CREATE, UPDATE, and DELETE. Optimized for business logic and consistency.
 2. **The Query Side:** A specialized, "read-only" database (a Read Model) that is a pre-joined, flattened version of the data.
### 7.3 Maintaining the Read Model
The Query Side is kept in sync by subscribing to **Domain Events** published by the Command Side. When an "OrderCreated" event occurs, a "Projector" updates the Read Model database (which could be Elasticsearch, MongoDB, or even a different SQL table).
## Technical Examples: Creating a Read Model
### C# Example: Projecting Events to a SQL Read Model
In .NET, you often use an IEventHandler that listens for events (perhaps via a library like MediatR) to update a flattened table optimized for the UI.
```csharp
// The 'Projector' that maintains a fast Read Model
public class OrderSummaryProjector : INotificationHandler<OrderCreatedEvent>
{
    private readonly ReadDbContext _context;

    public OrderSummaryProjector(ReadDbContext context) => _context = context;

    public async Task Handle(OrderCreatedEvent notification, CancellationToken ct)
    {
        // We "flatten" the data into a single row for a fast 'Orders List' view
        var summary = new OrderSummaryView
        {
            OrderId = notification.OrderId,
            CustomerName = notification.CustomerName,
            TotalAmount = notification.Total,
            Status = "Pending",
            CreatedAt = DateTime.UtcNow
        };

        _context.OrderSummaries.Add(summary);
        await _context.SaveChangesAsync(ct);
    }
}

```
### Python Example: Syncing to an Elasticsearch View
Python is frequently used for "Consumer" scripts that take events from a broker and push them into a search engine like Elasticsearch for high-performance filtering.
```python
from elasticsearch import Elasticsearch

es = Elasticsearch(["http://localhost:9200"])

def handle_order_event(event_type, data):
    if event_type == "ORDER_CREATED":
        # Create a document perfectly structured for the 'Order History' UI
        doc = {
            "order_id": data['id'],
            "status": "CREATED",
            "display_text": f"Order for {data['customer']}",
            "timestamp": data['created_at']
        }
        # Index into the 'Query Side' database
        es.index(index="orders_view", id=data['id'], document=doc)

    elif event_type == "ORDER_CANCELLED":
        # Update the Read Model accordingly
        es.update(index="orders_view", id=data['id'], doc={"status": "CANCELLED"})

```
## Key Takeaways
 1. **Separate Concerns:** The database schema that is best for *writing* (normalized) is rarely the best for *reading* (denormalized).
 2. **Eventual Consistency:** The Read Model may be slightly behind the Command Side (milliseconds). Use "Optimistic UI" updates to hide this from users.
 3. **Efficiency:** CQRS allows you to use the right tool for the job—e.g., using a Relational DB for transactions and Elasticsearch for complex 