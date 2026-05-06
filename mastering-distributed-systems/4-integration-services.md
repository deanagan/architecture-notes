# Chapter 4: Azure Integration Services (AIS)
In the Azure ecosystem, you often mix and match services depending on the **reliability** vs. **throughput** requirements.
### 4.1 Azure Service Bus vs. Event Hubs
This is the most common design question in Azure interviews.
 * **Event Hubs (The Firehose):** Designed for millions of events. Great for logging, telemetry, and high-speed data streams.
 * **Service Bus (The High-Value Courier):** Designed for **financial transactions** and **commands**. It supports complex features like:
   * **Scheduled Messages:** "Send this email in 2 hours."
   * **Peek-Lock:** A consumer "locks" a message; if the consumer crashes, the message automatically reappears for another consumer.
   * **Sessions:** Ensures strict FIFO (First-In-First-Out) ordering for related messages.
### 4.2 Azure Logic Apps: The Low-Code Orchestrator
Sometimes, writing a custom C# or Python consumer is overkill. **Logic Apps** allow you to create visual workflows that connect to hundreds of external services (Slack, Salesforce, Office 365, SQL Server) without writing boilerplate code.
**Common Pattern:**
Event Hub Event -> Azure Function (Data Cleanup) -> Service Bus Queue -> Logic App (Notify Manager via Email).
### 4.3 Implementation: Service Bus Queue Producer
Let's see how a "Command" (e.g., ProcessPayment) differs from an "Event" (e.g., OrderCreated).
**Tech Stack:** Python with azure-servicebus
```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "Endpoint=sb://your-namespace..."
QUEUE_NAME = "payment-commands"

def send_payment_command(payment_details):
    client = ServiceBusClient.from_connection_string(CONNECTION_STR)
    
    with client:
        sender = client.get_queue_sender(queue_name=QUEUE_NAME)
        # We wrap the data in a ServiceBusMessage
        message = ServiceBusMessage(payment_details)
        
        # Service Bus allows 'Scheduled' messages - very powerful!
        # sender.schedule_messages(message, enqueue_time_utc=...)
        
        sender.send_messages(message)
        print("Payment command sent to queue with Peek-Lock enabled.")

```
### 4.4 Serverless Orchestration with Azure Functions
When your "Consumer" logic gets complex (e.g., "Wait for 2 days, then check status"), standard Azure Functions get messy. This is where **Durable Functions** come in.
**Durable Functions Patterns:**
 1. **Function Chaining:** Execute A, then B, then C.
 2. **Fan-out/Fan-in:** Start 100 processes at once, then wait for all to finish before proceeding.
 3. **Human Interaction:** Pause the code for 24 hours until someone clicks "Approve" in an email.
### 4.5 Best Practices: The "Enterprise Integration" Way
 * **Use Queues for Commands:** If Service A tells Service B to "Do X," use a Service Bus Queue.
 * **Use Topics for Events:** If Service A says "I just did X" and doesn't care who listens, use Kafka or Event Hubs.
 * **Dead Lettering:** Always enable the Dead Letter Queue (DLQ) in Service Bus. It acts as a safety net for messages that can't be processed due to bad data or downstream outages.
