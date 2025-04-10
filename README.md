# Event Wagon Pattern

An architecture for managing explosive event flows using self-contained units called **wagons**, which detach from the main stream, process their backlog, and are later discarded or rotated. This pattern aims to preserve low latency and system resilience even under unpredictable loads, without overloading the global infrastructure.

---

## 🎯 Technical Motivation

In Kafka-based or event-driven systems, the most dangerous problems often aren’t failures — but slowdowns that grow silently.

- **Rising lag** affects all incoming events.
- **Blind horizontal scaling** adds consumers but doesn’t isolate the root issue.
- **Rollback is painful**, affecting live traffic.
- **No semantic or temporal grouping** makes auditing or debugging traffic spikes a nightmare.

The Event Wagon Pattern introduces a new approach: **temporally isolating event batches** into independent processing units, coordinated by lightweight infrastructure.

---

## 🧠 What is a Wagon?

A logical unit of events that:
- Receives a portion of the event stream.
- Detaches upon reaching a threshold (volume, lag, etc.).
- Gets processed in isolation with its own infrastructure.
- Is discarded or reused once completed.

---

## 🔁 Core Components

- **Switcher**: routes events to the current active wagon.
- **Observer**: monitors the wagon’s health and asks the switcher to create a new one when needed.
- **Wagons**: isolated processing units (topic + infra).
- **External orchestrator (optional)**: creates wagons on demand (K8s, AWS, etc.).

---

## ⚙️ Basic Flow

1. The system starts with an active wagon (`wagon_1`).
2. The switcher routes all events to it.
3. The observer detects saturation (lag, delay, etc.).
4. The observer notifies the switcher → create new wagon.
5. Switcher provisions or triggers infra for `wagon_2`.
6. Once ready, the switcher redirects incoming events.
7. The observer waits until `wagon_1` is done and cleans it up.

---

## 💡 Benefits

- Keeps low latency for new events
- Total isolation between event batches
- Easy rollback, testing, and traceability
- Compatible with Kafka, Pulsar, Redis Streams
- Supports rotation or reuse of wagons

---

## 📦 Repo Contents

- `pattern/`: Full theoretical framework
- `examples/`: Real use cases like Black Friday
- `prototype/`: Space for early MVPs (Go, Rust, etc.)
- `diagrams/`: Visual diagrams (coming soon)
- `AUTHOR.txt`: Author credits
- `LICENSE`: CC BY-NC 4.0 license

---

## ✨ Current Status

> The pattern is in its early documentation and public feedback stage.  
> Contributions and real-world testing are welcome.

---

## 🔗 Read more

👉 See `pattern/event_wagon_pattern.md` for the complete theoretical writeup.
