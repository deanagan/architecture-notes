# Managing Transactions with Sagas

# 4.1 The Challenge of Distributed Data

When a business process spans multiple services (e.g., placing an order, verifying credit, and reserving inventory), you cannot lock all those databases at once. If one step fails, you need a way to undo the previous steps.

# 4.2 The Saga Pattern

A Saga is a sequence of local transactions. Each local transaction updates the database and publishes a message or event to trigger the next local transaction in the saga.
 * Compensating Transactions: If a local transaction fails (e.g., credit card declined), the saga must execute a series of "undo" operations to restore consistency.
 * Saga Coordination:
   * Choreography: Each service listens for events and decides what to do next. Simple, but can become a "spaghetti" of dependencies.
   * Orchestration: A central "Saga Orchestrator" tells each service what to do. Easier to manage and understand for complex workflows.

Technical Examples: Implementing Saga Logic
C# Example: Orchestration with a State Machine
In .NET, libraries like MassTransit or Stateless are often used to manage the state of a long-running saga.

```csharp
// Simplified Orchestrator Logic using a State Machine approach
public class OrderSaga : MassTransitStateMachine<OrderState>
{
    public OrderSaga()
    {
        InstanceState(x => x.CurrentState);

        Initially(
            When(OrderSubmitted)
                .Then(context => context.Instance.OrderId = context.Data.OrderId)
                .TransitionTo(VerifyingCredit)
                .Publish(context => new VerifyCredit(context.Instance.OrderId))
        );

        During(VerifyingCredit,
            When(CreditApproved)
                .TransitionTo(ReservingStock)
                .Publish(context => new ReserveStock(context.Instance.OrderId)),
            When(CreditRejected)
                .Then(context => Console.WriteLine("Compensating: Cancelling Order"))
                .TransitionTo(Cancelled)
        );
    }
}
```

### Python Example: Event-Driven Compensating Transaction

In Python, using an event bus (like NATS or Kafka), you might implement a simple "Choreography" where a failure triggers a rollback event.

```python
# inventory_service.py
def handle_reserve_stock(event):
    try:
        # Business logic to reserve stock
        if not stock_available(event['item_id']):
            raise Exception("Out of stock")
        
        reserve_items(event['item_id'])
        emit_event("STOCK_RESERVED", {"order_id": event['order_id']})
        
    except Exception as e:
        # Trigger the compensating transaction in previous services
        emit_event("STOCK_FAILED", {
            "order_id": event['order_id'],
            "reason": str(e)
        })

# order_service.py (Listening for failures)
def handle_stock_failed(event):
    print(f"Rolling back order {event['order_id']}: {event['reason']}")
    update_order_status(event['order_id'], "CANCELLED")
```

Key Takeaways

 * Embrace Eventual Consistency: Your system won't be consistent every millisecond, but the Saga ensures it is consistent eventually.
 * Lack of Isolation: Sagas lack the "I" (Isolation) in ACID. You must use countermeasures like "semantic locks" (marking a record as Pending) to prevent other transactions from using dirty data.
 * Design for Failure: Every step in a Saga must have a corresponding "Undo" step.
