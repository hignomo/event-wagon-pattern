
# Event Wagon Pattern

**An architectural pattern based on the temporary isolation of event flows to maintain low latency and high resilience during unpredictable traffic peaks.**

---

## 1. Introduction

The *Event Wagon Pattern* was born as a response to a common problem in modern distributed systems: progressive saturation of event channels leading to growing latency, uncontrollable backlogs, and inefficient scaling.

This pattern proposes dividing the event flow into self-contained blocks called **wagons**, which fill up, decouple from the main flow, process their backlog in isolation, and are then discarded or rotated. Each wagon contains its own event choreography.

---

## 2. Problem it addresses

- Sudden spikes in event traffic (mass signups, viral traffic, IoT alerts).
- Linear processing pipelines collapsing under unpredictable loads.
- Ineffective and expensive reactive scaling.
- Rising latency for new users during backlog.
- Difficulty auditing, tracing, or debugging by event batch.

---

## 3. Core concept

### What is a **wagon**?

A logical unit of event processing that:

- Receives a limited portion of the event flow (by time, volume or lag).
- Detaches from the ingestion stream when considered “full”.
- Processes its backlog in isolation with its own infrastructure.
- Is removed or reused once finished.

### Pattern components:

- **Switcher:** routes events to the active wagon.
- **Observer:** monitors active wagons and detects when a new one is needed.
- **Wagons:** self-contained topics or channels with dedicated consumers.
- **Orchestrator (optional):** provisions infrastructure when triggered by the switcher.

---

## 4. How it works

1. A wagon (`wagon_1`) is active, receiving all events.
2. The *Switcher* routes ingestion to `wagon_1`.
3. When thresholds are reached (lag, throughput), the *Observer* notifies the *Switcher*.
4. The *Switcher* provisions `wagon_2` (or instructs the orchestrator).
5. Once `wagon_2` is ready, ingestion is routed to it.
6. `wagon_1` processes its backlog independently.
7. The *Observer* removes `wagon_1` once it's empty.
8. The cycle repeats as needed.

---

## 5. Advantages

- **Full isolation** between past and current event flows.
- **Precise control** over processing by batches.
- **Guaranteed low latency** for recent users.
- **Easier testing, rollback, and auditability.**
- **Ideal for serverless or ephemeral infrastructure.**
- **Compatible with Kafka, Pulsar, Redis Streams, and others.**

---

## 6. Limitations

- Requires coordination between components (switcher, observer).
- Not suited for workflows with strict event ordering.
- Increases number of topics and monitoring points.
- Needs well-defined lifecycle logic.

---

## 7. Basic example

On Black Friday, an e-commerce receives 15,000 user signups in 2 minutes:

- `wagon_1` (topic `user_registrations_v1`) receives the first 10,000 events.
- Lag starts increasing → the observer signals to switch.
- `wagon_2` is created, and new events are routed there.
- `wagon_1` continues processing in isolation → eventually discarded.
- The system keeps functioning with low latency for new users.

---

## 8. Recommended use cases

- Unpredictable load surges in e-commerce.
- Concurrent logins in enterprise systems.
- IoT or telemetry signal bursts.
- Massive notification or email pushes.
- Asynchronous event pipelines with decoupled processors.

---

## 9. Comparison with other solutions

| Solution                           | Isolation | Backlog control | Low-latency under pressure |
|------------------------------------|-----------|------------------|-----------------------------|
| Scaling consumers/partitions       | ❌        | Low              | Low                        |
| Serverless + queue                 | ⚠️ Partial | Medium           | Medium                     |
| Event Wagon Pattern                | ✅        | **High**         | **Very high**              |

---

## 10. Integration with orchestrators

In environments like Kubernetes or AWS, the Switcher does not create the wagons directly, but sends a request to an orchestrator. The orchestrator handles the provisioning of topics and consumers, and when ready, returns a `ready` signal to the Switcher. Only then is ingestion redirected.

---

## 11. Comparisons and relevance

There are partial solutions in the ecosystem, such as dynamic Kafka scaling, retry logic, or stream segmentation — but none provide a formalized, named, and reusable architecture like this pattern. It offers a balance between **control, scalability, and clarity**, while being stack-agnostic and cloud-friendly.

---

## 12. Why this pattern matters

By turning event streams into **rotating, disposable wagons**, this pattern allows:

- Lower latency for new traffic
- Isolation of risk and backlog
- Improved monitoring and cleanup
- Cost-effective resource use

It’s not about replacing Kafka — it’s about **helping it breathe** under pressure.

---

## 13. Author and license

**Original Author:** Alfonso Jose Navarro Escámez  
**Date of creation:** April 2025  
**License:** Creative Commons BY-NC 4.0

---

## ✨ Final note

> This is a contribution, not a claim to ownership.  
> If it helps you or sparks something better — I’ll consider it a success.

---
