## Chapter 1: The Modern RESTful API
Before we introduce the complexity of message brokers like Kafka, the backbone of any system is a reliable, predictable API. We will focus on building a robust entry point for your services.
### 1.1 Clean Architecture & Layering
A common mistake is "Leaky Abstractions," where database logic or business rules bleed into the controller. We use a layered approach:
 1. **API/Controller Layer:** Handles HTTP requests and routing.
 2. **Service/Domain Layer:** Contains the "Why" (Business Logic).
 3. **Infrastructure/Persistence Layer:** Handles the "How" (Database/External APIs).
### 1.2 Best Practices: The Checklist
 * **Use Nouns, Not Verbs:** /api/v1/orders instead of /api/v1/getOrders.
 * **Version Everything:** Always use /v1/ to avoid breaking changes for consumers.
 * **Idempotency:** Ensure that calling a PUT or DELETE multiple times results in the same state.
 * **Standardized Error Responses:** Don't just return a 500; return a structured body.
### 1.3 Implementation Sample (Python/FastAPI)
Below is a pattern for a scalable Ledger entry endpoint. Note the separation of the **Model** (Database) from the **Schema** (Request/Response).
```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from typing import List, Optional
from uuid import UUID, uuid4

app = FastAPI(title="Fintech Ledger API", version="1.0.0")

# --- Schemas (DTOs) ---
class TransactionBase(BaseModel):
    amount: float
    currency: str
    description: str

class TransactionCreate(TransactionBase):
    account_id: UUID

class TransactionResponse(TransactionBase):
    id: UUID
    status: str = "PENDING"

# --- Mock Service Layer ---
@app.post("/v1/transactions", 
          response_model=TransactionResponse, 
          status_code=status.HTTP_201_CREATED)
async def create_transaction(transaction: TransactionCreate):
    # logic: check account balance, verify currency
    if transaction.amount <= 0:
        raise HTTPException(status_code=400, detail="Amount must be positive")
    
    return {
        **transaction.dict(),
        "id": uuid4(),
        "status": "PROCESSED"
    }

```
### 1.4 API Design Principles
 * **201 Created:** Use for successful POSTs.
 * **400 Bad Request:** Use for validation errors (e.g., negative amounts).
 * **401 vs 403:** 401 is "I don't know who you are"; 403 is "I know you, but you aren't allowed here."
 * **429 Too Many Requests:** Essential for rate limiting in high-load systems.

**Next Steps for Chapter 2:**
We will take this API and move from **Synchronous** processing to **Asynchronous** streaming. Instead of the API waiting for a database write, it will fire an event to **Azure Event Hubs** or **Kafka** and return immediately.
Would you like to dive deeper into specific API security patterns (like OAuth2/OpenID Connect) before we move to Kafka?
