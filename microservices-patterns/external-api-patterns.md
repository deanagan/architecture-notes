# External API Patterns
Since services are now decomposed and queries are split, how do external clients (Mobile apps, Web apps, Third-party APIs) interact with the system?
### 8.1 The Problem with Direct Client-to-Service Communication
If a mobile app talks to 20 different microservices directly:
 * **Too many requests:** Over a mobile network, 20 round-trips are slow.
 * **Tight coupling:** If you rename a service or split one into two, every client app has to be updated.
 * **Security/Protocol issues:** Some services might use gRPC or AMQP, which aren't friendly for browser/mobile clients.
### 8.2 The API Gateway Pattern
An **API Gateway** acts as a single entry point for all clients. It handles:
 * **Request Routing:** Sending the request to the correct service.
 * **API Composition:** Performing the "Chapter 7" join in one go so the mobile app only makes one request.
 * **Cross-cutting concerns:** Authentication, SSL termination, rate limiting, and caching.

(cross)[cross.jpg]

### 8.3 The Backends for Frontends (BFF) Pattern
A variation where you have a specific gateway for each type of client (e.g., one for the iOS app, one for the Web app). This prevents one massive "monolithic" gateway.
## Technical Examples: Gateway Logic
### C# Example: Ocelot Gateway Configuration
In .NET, **Ocelot** is a popular library for building API Gateways. Most of the logic is defined in a JSON configuration file.
```json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/orders/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [
        { "Host": "order-service", "Port": 80 }
      ],
      "UpstreamPathTemplate": "/orders/{everything}",
      "UpstreamHttpMethod": [ "Get", "Post" ]
    }
  ]
}

```
### Python Example: Reverse Proxy with FastAPI
You can build a simple gateway "orchestrator" in Python that aggregates data from multiple internal services.
```python
import httpx
from fastapi import FastAPI

app = FastAPI()

@app.get("/order-summary/{order_id}")
async def get_order_summary(order_id: str):
    async with httpx.AsyncClient() as client:
        # Concurrent calls to multiple internal services
        order_task = client.get(f"http://order-service/orders/{order_id}")
        shipping_task = client.get(f"http://shipping-service/shipments/{order_id}")
        
        # Aggregate results
        order_res, shipping_res = await asyncio.gather(order_task, shipping_task)
        
        return {
            "order": order_res.json(),
            "shipping": shipping_res.json()
        }

```
**Key Takeaway:** The Gateway simplifies the client's life by hiding the complexity of the microservice 