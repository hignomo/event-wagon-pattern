# Use Case: Black Friday

An e-commerce platform receives 15,000 user registrations within 2 minutes:

1. **Wagon_1** is active and receives the first batch of events.
2. Once a defined threshold is reached (e.g. lag or count), the **Observer** detects saturation.
3. The Observer notifies the **Switcher** to create **Wagon_2**.
4. The Switcher provisions and redirects new incoming events to Wagon_2.
5. **Wagon_1** continues processing its backlog in isolation.
6. When Wagon_1 is empty, the **Observer** terminates it.
7. Wagon_2 becomes the only active ingestion unit until the next rotation.

