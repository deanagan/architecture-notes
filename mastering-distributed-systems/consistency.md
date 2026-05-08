This is where most backend systems fail. When you move from a single database to multiple microservices, you lose "ACID" transactions. You can no longer wrap a database write and a Kafka message send in one single block.
If the database saves but the Kafka message fails, your system is **inconsistent**.
## Chapter 5: Data Consistency in Microservices
In this chapter, we master the patterns that ensure your data remains the "Single Source of Truth" across the entire Azure ecosystem.
### 5.1 The Dual-Write Problem
Imagine you are building a banking app.
 1. Your API saves a "Withdrawal" to the **SQL Database**.
 2. Your API then tries to send a "BalanceUpdated" event to **Kafka**.
 3. **The network dies.**
Now, the user has lost money in the database, but the "Email Service" and "Analytics Service" never got the message. This is the **Dual-Write Problem**.
### 5.2 The Transactional Outbox Pattern
This is the gold standard for solving the dual-write problem.
 * **The Move:** Instead of sending the message to Kafka directly, you save the message into a special table in your **same database** called the Outbox.
 * **The Guarantee:** Since the Business Data and the Outbox Message are in the same database, they are part of a single, atomic transaction.
 * **The Relay:** A separate background process (like an Azure Function or a tool called **Debezium**) reads the Outbox table and pushes those messages to Kafka.
### 5.3 Implementation: The Outbox Pattern (SQL + Worker)
**Step 1: The Atomic Save (Python/SQLAlchemy)**
```python
def create_order(db_session, order_data):
    # 1. Save the Order
    new_order = Order(**order_data)
    db_session.add(new_order)
    
    # 2. Save the Event to the Outbox table in the SAME transaction
    outbox_event = Outbox(
        aggregate_type="Order",
        payload=json.dumps(order_data),
        status="PENDING"
    )
    db_session.add(outbox_event)
    
    # Commit both or neither!
    db_session.commit()

```
**Step 2: The Message Relay (Azure Function)**
A Timer-triggered function runs every second, picks up "PENDING" events, sends them to Kafka, and marks them as "PROCESSED."
### 5.4 Idempotency with CosmosDB
On the **Consumer** side, you must assume you will receive the same message twice. We use a **"Check-and-Set"** pattern.
Azure CosmosDB is perfect for this because of its **ETag** or **Point Reads**.
**The Logic:**
 1. Consumer receives Order_123.
 2. Consumer checks CosmosDB: SELECT * FROM ProcessedEvents WHERE id = 'Order_123'.
 3. If it exists, **Discard** (return 200).
 4. If not, **Process** the order and **Insert** the ID into CosmosDB.
### 5.5 Distributed Locking (Azure Redis)
Sometimes you need to ensure only one worker processes a specific resource at a time.
 * **Problem:** Two consumers read the same partition and both try to update a user's gold balance.
 * **Solution:** Use **Redlock** (Redis Distributed Lock). The worker must "acquire" a lock in Redis before updating.
```python
import redis

r = redis.Redis(host='your-azure-redis.cache.windows.net', port=6380)

def process_with_lock(user_id):
    lock = r.lock(f"lock:user:{user_id}", timeout=5)
    if lock.acquire(blocking=False):
        try:
            # Perform sensitive balance update here
            pass
        finally:
            lock.release()
    else:
        print("Another worker is already processing this user.")

```
### 5.6 Best Practices: Consistency Checklist
 * **Prefer Eventual Consistency:** Don't try to make the whole system "Instant." Accept that the Email Service might be 2 seconds behind the Database.
 * **Database-per-Service:** Never let two services share the same database. If they need to share data, use Kafka.
 * **Poison Pill Handling:** If a message is consistently causing a crash, use the **Dead Letter Queue (DLQ)** pattern we discussed in Chapter 4.
**We’ve built a robust, consistent engine. Now we need to know if it's actually working. Ready for Chapter 6: Observability & Resilience (Tracing, Circuit Breakers, and App Insights)?**
