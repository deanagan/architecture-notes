Once your distributed system is running, the biggest challenge is "Where did this request go?" In a monolith, you check one log file. In a distributed Azure environment, a single user click might touch an API Gateway, a FastAPI service, a Kafka topic, and three different Azure Functions.
## Chapter 6: Observability & Resilience
This chapter covers how to monitor complex flows and how to prevent a single failing service from taking down your entire architecture.
### 6.1 Distributed Tracing with Application Insights
To track a request across multiple services, we use a **Correlation ID**. In Azure, **Application Insights** handles this automatically using the W3C Trace Context.
 * **Trace ID:** Follows the request from the front door to the final database write.
 * **Span ID:** Tracks the time spent inside one specific service or function.
**Implementation Tip:** When you produce a Kafka message, you must "inject" the Trace ID into the Kafka **Message Headers**. The consumer then "extracts" it so the logs remain linked.
### 6.2 The Circuit Breaker Pattern
In a distributed system, if "Service B" is down, "Service A" should stop trying to call it immediately. If it keeps trying, it wastes resources and creates a bottleneck.
**The Three States:**
 1. **Closed:** Everything is normal. Requests flow through.
 2. **Open:** The external service is failing. The Circuit Breaker "trips" and returns an error immediately without calling the service.
 3. **Half-Open:** After a timeout, the breaker allows a *few* requests through to see if the service has recovered.
### 6.3 Implementation: Resilience with Resilience4J or Tenacity
For Python backends, the tenacity or pybreaker libraries are common. In C#, Polly is the industry standard.
```python
import pybreaker

# Define the breaker: fail after 5 consecutive errors, 
# then wait 60 seconds before trying again.
db_breaker = pybreaker.CircuitBreaker(fail_max=5, reset_timeout=60)

@db_breaker
def call_external_service():
    # This logic is now protected
    response = requests.get("https://api.external-partner.com/data")
    return response.json()

```
### 6.4 Health Checks & Probes
Azure Container Apps and Kubernetes use **Probes** to decide if your service is healthy:
 * **Liveness Probe:** "Are you alive?" (If no, restart the container).
 * **Readiness Probe:** "Are you ready to handle traffic?" (If no, stop sending requests here, but don't restart yet).
**Best Practice:** Don't just return 200 OK. Your health check should verify its connection to the database and Kafka.
## 6.5 The "Golden Signals" of Monitoring
When looking at your Azure Dashboards, focus on these four:
 1. **Latency:** How long does it take to process a request/event?
 2. **Traffic:** How many requests per second?
 3. **Errors:** What is the rate of 5xx errors?
 4. **Saturation:** How "full" is your service? (CPU, Memory, Kafka Partition lag).
## Chapter 7: System Design: Scaling to Millions
We have reached the final chapter. Now we look at the "Big Picture" design.
### 7.1 Horizontal vs. Vertical Scaling
 * **Vertical:** Adding more RAM/CPU to one machine (Expensive and has a limit).
 * **Horizontal:** Adding more instances of your service (The Cloud way).
### 7.2 Database Sharding & Partitioning
When your SQL database gets too big, you split it.
 * **Vertical Partitioning:** Putting "User Profiles" in one DB and "User Orders" in another.
 * **Horizontal Partitioning (Sharding):** Putting Users A-M in Database 1 and Users N-Z in Database 2.
### 7.3 Global Load Balancing: Azure Front Door
For a truly global system, you use **Azure Front Door**. It uses **Anycast IP** to route users to the nearest Azure region, reducing latency and providing Web Application Firewall (WAF) protection at the edge.
### Final Best Practices Summary
 1. **Statelessness:** Your API instances should not store local data. Use Redis or a Database.
 2. **Infrastructure as Code (IaC):** Use Bicep or Terraform to deploy your Azure services.
 3. **Security First:** Use Managed Identities so your code doesn't need to store connection strings or passwords.
