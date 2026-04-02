# Inter-Process Communication in a Microservice Architecture

Since microservices turn a monolithic "in-memory" call into a distributed system, communication becomes a central architectural concern.

## 3.1 Overview of Communication Styles
Richardson categorizes IPC based on two dimensions:
 * One-to-one vs. One-to-many: Does a request go to a single service or multiple?
 * Synchronous vs. Asynchronous: Does the client wait for a response or send and forget?

| Style | Interaction Type | Examples |
|---|---|---|
| One-to-One | Request/Response | REST, gRPC, Thrift |
| One-to-One | Asynchronous Request/Response | Messaging (Command/Reply) |
| One-to-Many | Publish/Subscribe | Kafka, RabbitMQ, SNS |

## 3.2 Evolution of APIs
Because services change independently, API Versioning is critical.
 * Semantic Versioning (MAJOR.MINOR.PATCH): A standard way to signal breaking vs. non-breaking changes.
 * Consumer-Driven Contract Testing: Ensuring that a change in the "Provider" service doesn't break the "Consumer" service.
3.3 Message-Based Communication
Using a message broker (like RabbitMQ or Kafka) provides Loose Coupling and Increased Availability. If the receiver is down, the message stays in the queue.
Technical Examples: Synchronous (gRPC) and Asynchronous (Messaging)
C# Example: gRPC (Synchronous)
C# and .NET have excellent support for gRPC, which uses Protocol Buffers for high-performance, strongly-typed communication.
// Define the service in a .proto file
/*
service OrderService {
  rpc GetOrderDetails (OrderRequest) returns (OrderResponse);
}
*/

// Implementation in C#
public class OrderGrpcService : OrderService.OrderServiceBase
{
    public override async Task<OrderResponse> GetOrderDetails(
        OrderRequest request, ServerCallContext context)
    {
        var order = await _db.Orders.FindAsync(request.OrderId);
        return new OrderResponse 
        { 
            OrderId = order.Id.ToString(), 
            Status = order.Status 
        };
    }
}

Python Example: RabbitMQ (Asynchronous)
Python is frequently used for background workers that process messages from a queue using libraries like pika.
import pika
import json

def process_order_created(ch, method, properties, body):
    data = json.loads(body)
    print(f" [x] Processing order: {data['order_id']}")
    # Perform background task (e.g., send confirmation email)
    ch.basic_ack(delivery_tag=method.delivery_tag)

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.queue_declare(queue='order_placed_queue')
channel.basic_consume(queue='order_placed_queue', on_message_callback=process_order_created)

print(' [*] Waiting for messages. To exit press CTRL+C')
channel.start_consuming()

Key Takeaways
 * Prefer Asynchronous when possible: It improves system resiliency and decouples the availability of services.
 * Service Discovery: Use patterns like Client-side discovery or Server-side discovery (e.g., Consul, Kubernetes DNS) so services can find each other without hardcoded IP addresses.
 * Handle Partial Failure: Use the Circuit Breaker pattern to prevent a failure in one service from cascading and taking down the entire system.
