# Chapter 1 – Reliable, Scalable, and Maintainable Applications

This chapter shifts the focus from "databases" as a single category to Data Systems. In modern development, we often combine multiple tools (caches, indexes, message queues) to build a functional system. Kleppmann argues that a data-intensive application is defined by three core concerns.

## 1. Reliability

Definition: The system should continue to work correctly even when things go wrong.
 * Fault vs. Failure: A fault is one component deviating from its spec; a failure is the entire system stopping service. We build fault-tolerant systems to prevent faults from triggering failures.
 * Hardware Faults: Hard disks crash, RAM becomes corrupt. Solutions involve redundancy (RAID, dual power supplies, failover clusters).
 * Software Errors: Harder to anticipate. Examples include a process consuming shared resources (CPU/RAM) or a library having a cascading bug.
 * Human Error: The leading cause of outages. Mitigate this through well-defined abstractions, "sandbox" environments for testing, and detailed monitoring.

## 2. Scalability

Definition: The system's ability to cope with increased load. Scaling is not a "yes/no" feature; it's a question of "How do we stay reliable if the load increases?"
Describing Load
We use load parameters to measure specific pressures on a system:
 * Requests per second to a web server.
 * The ratio of reads to writes in a database.
 * The number of simultaneous active users.
Describing Performance
When load increases, we look at how performance is affected.
 * Throughput: The number of records processed per second (common in batch systems).
 * Response Time: The time between a client sending a request and receiving a response.
 * Percentiles (p95, p99, p99.9): Averages are misleading. The p99 (99th percentile) tells you how the slowest 1\% of your users are experiencing the system, which is critical for identifying "tail latency."
Scaling Architecture
 * Scaling Up (Vertical): Buying a more powerful machine.
 * Scaling Out (Horizontal): Distributing the load across many smaller machines.
 * Elastic Systems: Automatically add computing resources when they detect a load increase.

## 3. Maintainability

Definition: The majority of the cost of software is not in its initial development, but in its ongoing maintenance.

| Principle | Goal |
|---|---|
| Operability | Make it easy for operations teams to keep the system running smoothly (good monitoring, documentation, and default behaviors). |
| Simplicity | Managing complexity by using abstractions. A clean API allows you to hide a complex implementation behind a simple interface. |
| Evolvability | Make it easy for engineers to make changes in the future (also known as extensibility or agility). |

Summary Table: The Three Pillars
| Pillar | Focus | Key Strategy |
|---|---|---|
| Reliability | Correctness | Redundancy and Error Handling |
| Scalability | Performance | Load Parameters and Horizontal Scaling |
| Maintainability | People | Abstraction and Operability |
