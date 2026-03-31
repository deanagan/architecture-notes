# Chapter 2: Decomposition Strategies

Chapter 2 focuses on the most critical challenge of microservices: How do you decide which services to create? Richardson argues that the success of your architecture depends on how well you decompose the application into services.

## 2.1 Decomposition Patterns

There are two primary patterns for defining your microservice boundaries:
 * Decompose by Business Capability: Identify services based on what the business does (e.g., Order Management, Customer Management, Delivery). This is often stable as business functions change slowly.
 * Decompose by Subdomain (DDD): Use Domain-Driven Design (DDD) concepts. You identify "Bounded Contexts" within the overall problem domain. Each sub-domain (Core, Supporting, or Generic) becomes a candidate for a microservice.

## 2.2 Obstacles to Decomposition

Decomposition isn't just about drawing lines; you must manage:
 * Network Latency: Too many fine-grained services lead to "chatty" communication.
 * Synchronous Communication: Reducing availability if one service goes down.
 * Data Consistency: Maintaining "Sagas" (Chapter 4) across service boundaries.
 * The "God Class": Avoiding giant, monolithic classes (like a User or Product object) that are used everywhere but mean different things in different contexts.

Technical Examples: Handling Shared Data (The "God Class" Problem)
In a Monolith, you might have one giant Order class. In Microservices, you decompose it. The "Ordering" service sees an Order as a list of items; the "Shipping" service sees it as a delivery address and a tracking number.
### C# Example: Specific Models per Service
In C#, we use separate namespaces or even separate projects to ensure that the Order model in the Shipping Service does not share code with the Sales Service.

```csharp
// In Shipping.Service.Models
namespace Shipping.Service.Models
{
    public class Order
    {
        public Guid OrderId { get; set; }
        public string ShippingAddress { get; set; }
        public DeliveryStatus Status { get; set; }
        // Note: No 'LineItems' or 'Price' here; Shipping doesn't care.
    }
}

// In Sales.Service.Models
namespace Sales.Service.Models
{
    public class Order
    {
        public Guid OrderId { get; set; }
        public List<LineItem> Items { get; set; }
        public decimal TotalPrice { get; set; }
        // Note: No 'ShippingAddress' here; Sales doesn't care.
    }
}
```

### Python Example: Decoupled Logic (Pydantic)

Using Python's type hinting and Pydantic, we can define lightweight schemas that represent only the slice of data relevant to a specific domain.

```python
# shipping_service/schemas.py
from pydantic import BaseModel
from uuid import UUID

class ShippingOrder(BaseModel):
    order_id: UUID
    tracking_number: str
    destination: str

# sales_service/schemas.py
from pydantic import BaseModel
from typing import List
from uuid import UUID

class SalesOrder(BaseModel):
    order_id: UUID
    items: List[str]
    total_amount: float
```

Key Takeaways

 * Loose Coupling is King: If you have to change two services to implement one business requirement, they are too tightly coupled.
 * Encapsulation: Each service should own its data. No "Shared Database" (which is a common anti-pattern).
 * High Cohesion: Each service should have a small set of strongly related responsibilities.
