## Designing Business Logic in a Microservice Architecture

Chapter 5 focuses on how to structure the code *inside* a service. Richardson argues that the complex logic of a microservice requires a different approach than the "Transaction Script" pattern often found in simple applications.

### 5.1 Business Logic Patterns
The chapter compares two main ways to organize business logic:
 * **Transaction Script:** A procedural approach where a single method handles all the logic for a request. This works for simple services but becomes unmanageable as complexity grows.
 * **Domain Model:** An object-oriented approach where logic is distributed across a network of collaborating objects. This is the heart of **Domain-Driven Design (DDD)**.
### 5.2 The DDD Aggregate Pattern
In a microservice, a "God Class" or a massive interconnected object graph is a disaster. Richardson advocates for **Aggregates**:
 * An Aggregate is a cluster of domain objects that can be treated as a single unit.
 * Every Aggregate has a **Root Entity** (the only part external services or other aggregates can reference).
 * **Rule:** A single transaction can only create or update **one** aggregate. To update others, you must use domain events (Sagas).
## Technical Examples: Implementing Aggregates and Domain Events
### C# Example: Aggregate Root with Encapsulated Logic
In C#, we use private setters and public methods to ensure the "Aggregate Root" maintains total control over its internal state, preventing "Anemic Domain Models."
```csharp
public class Order : IAggregateRoot // Marker interface
{
    public Guid Id { get; private set; }
    private List<OrderLineItem> _items = new();
    public OrderStatus Status { get; private set; }

    // Business logic is inside the domain object, not the service layer
    public void AddItem(Guid productId, int quantity)
    {
        if (Status != OrderStatus.Created)
            throw new InvalidOperationException("Cannot add items to a confirmed order.");
            
        _items.Add(new OrderLineItem(productId, quantity));
    }

    public void Confirm()
    {
        Status = OrderStatus.Confirmed;
        // Logic to record a domain event would go here
    }
}

```
### Python Example: Decoupled Business Logic
Using Python's @property and custom exceptions, we can create a clean domain model that enforces business invariants.
```python
class Restaurant:
    def __init__(self, restaurant_id, name, menu):
        self.id = restaurant_id
        self.name = name
        self.menu = menu
        self.is_active = True

    def create_order(self, items):
        if not self.is_active:
            raise RestaurantNotAcceptingOrders(self.id)
            
        # Validate that all items belong to the menu
        self._validate_menu_items(items)
        
        return Order(restaurant_id=self.id, items=items)

    def _validate_menu_items(self, items):
        # Internal validation logic
        pass

```

## Key Takeaways
 1. **Avoid Anemic Domain Models:** Logic should live in your entities and value objects, not just in "Service" classes.
 2. **Boundary Power:** Using Aggregates makes it clear where a transaction begins and ends.
 3. **Domain Events:** When an aggregate changes state (e.g., OrderCreated), it should emit an event so other parts of the system (or other microservices) can react.


