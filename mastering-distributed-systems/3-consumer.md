# Chapter 3: Event-Driven Architecture (EDA) & Consumers
In this chapter, we focus on how to build resilient workers that pull data from Kafka/Event Hubs. We will explore the **Compensating Transaction** pattern and how to ensure you don't process the same order twice.
### 3.1 Consumer Groups: Scaling the Workload
One of the most powerful features of Kafka and Event Hubs is the **Consumer Group**.
 * If you have one consumer, it reads from all partitions.
 * If you add a second consumer to the same group, Kafka automatically splits the partitions between them.
### 3.2 The Idempotency Challenge
In distributed systems, **"exactly-once"** delivery is the holy grail but is notoriously difficult. Usually, you get **"at-least-once"** delivery.
 * **The Risk:** A network glitch happens after a consumer processes an order but *before* it can tell Kafka "I'm done." Kafka will send that order again.
 * **The Solution:** Your consumer must be **Idempotent**. It should check a database (like Redis or CosmosDB) to see if Order_123 was already processed before doing it again.
### 3.3 Implementation: The Azure Function Consumer
Azure Functions make consuming from Event Hubs incredibly simple via "Triggers." You don't have to write the polling logic; Azure handles it for you.
**Tech Stack:** C# (Azure Functions)
```csharp
[FunctionName("ProcessOrderEvent")]
public static async Task Run(
    [EventHubTrigger("orders", Connection = "EventHubConnectionString")] EventData[] events,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        try
        {
            string messageBody = Encoding.UTF8.GetString(eventData.Body.Array);
            var order = JsonConvert.DeserializeObject<Order>(messageBody);

            // 1. Idempotency Check
            if (await IsAlreadyProcessed(order.Id)) continue;

            // 2. Business Logic (e.g., Update Inventory)
            await _inventoryService.ReserveStock(order.ProductId, order.Quantity);

            log.LogInformation($"Order {order.Id} processed successfully.");
        }
        catch (Exception ex)
        {
            // 3. Error Handling (More on this in Chapter 5)
            log.LogError($"Failed to process event: {ex.Message}");
        }
    }
}

```
### 3.4 Advanced Pattern: The Saga Pattern (Choreography)
When you have multiple services (Inventory, Payment, Shipping), how do you ensure they all succeed or all fail? Since we don't have global "database transactions" in microservices, we use a **Saga**.
In **Choreographed EDA**, there is no central "boss." Services talk to each other via Kafka:
 1. **Order Service** emits OrderCreated.
 2. **Payment Service** hears OrderCreated, charges the card, and emits PaymentSuccessful.
 3. **Inventory Service** hears PaymentSuccessful and reserves the item.
**What if Payment fails?**
The Payment Service emits PaymentFailed. The Order Service hears this and marks the order as CANCELLED. This is a **Compensating Transaction**.
### 3.5 Best Practices for Consumers
 * **Parallelism:** Match your partition count to your maximum expected number of consumer instances.
 * **Batching:** Process events in batches (like the EventData[] array above) to reduce database round-trips.
 * **Dead Letter Queues (DLQ):** If a message fails 3 times, move it to a separate "poison" queue for manual inspection. Don't let it block the whole pipe!


