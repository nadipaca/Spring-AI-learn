## Kafka - Notes
In backend system kafka work as a central event pipeline between services. So, when a record updated or transaction initiated then the publisher publishes an event to a Kafka topic instead of
calling other services directly. Kafka stores these events reliably and distribute them partitions to make a system fast and scalable.

Backend services subscribe as consumer and process events independently. So, if a service slow down or restarts, kafka keeps the data safe until it's ready again. So, this simple flow system helps 
to build loosely coupled, resilient and ready for real-time workloads.

**Kafka is designed for failure**

## Power of Schema registry:
[proto_client](https://github.com/nadipaca/patient_management_system/blob/main/patient-service/src/main/java/com/pm/patientservice/grpc/BillingServiceGrpcClient.java), 
[protobuf](https://github.com/nadipaca/patient_management_system/blob/main/patient-service/src/main/proto/billing_service.proto)

In microservices data is a contract. What happens when a contract breaks?

### **Scenario:** Producer adds a new field or changes a data type. Suddenly, downstream consumer
crashes with a deserialization error. We call this "downstream drama"

The schema registry is a single source of truth that ensures producers and consumers are always
speaking the same language.

### Problems it solves:
* **Breaking Changes:** It ensures Compatibility rules (Backward, Forward or Full). If a producer tries to push a
schema that would break existing consumers, the registry rejects it immediately.
* **Payload Bloat:** Instead of sending entire schema (JSON) with every single message, the
producer only sends a tiny 5-byte Schema ID. This significantly reduces network bandwidth and storage costs.
* **Data Quality:** It prevents garbage data entering to our topics. If it doesn't match the 
registereed schema, it wont allow it in.

### How it works?
* **The Contract:** We define a schema using Avro, protobuf, or JSON schema.
* **The ID:** Producer sends back a schema to registry and get backs a unique ID.
* **The message:** The message is sent to kafka with that ID prefixed to the data.
* **The Lookup:** The consumer sees the ID, fetches the coorect schema from the registry(and caches it)
and safely desirializes the message.

## Bottom Line:
* Without a Schema Registry, our kafka topics are just "bucket of bytes", where anything can happen.
* With it, we have a strictly governed data pipeline where teams can evolve their services independently
without any fear of breaking the system.

## How real systems handles Idempotency, Failures and Offsets:
* Kafka is not just a messaging system.
* It is the backbone of modern event-driven architectures where reliability, scalability, and Fault tolerance are critical.
* In real backend system, three things matter the most:
  - Idempotency
  - Failure Handling
  - Offset monitoring

### Why Idempotency is critical in event-driven systems?
* In distributed systems, failures are normal:
  - network timeouts
  - producer retries
  - Consumer crashes
  - Pod restarts
* So, the same event can be delievered more than once.
* A kafka consumer must be idempotent (Processing the same event multiple times should not change the result)
* For eg: If the event is related to payment processing, then due to retries money can be deducted two times.

* **Common Approach**
  -  Store eventID in Redis or DB.
  -   If event already processed -> skip
  -   Otherwise -> process and save.
  
* **Main use cases:**
  - Payment Processing
  - Order Processing
  - Notification Pipelines
    
### How failures are handled in kafka-based event systems:
1.  **Kafka always assumes consumer will fail. so, if consumer crashes while processing:**
- kafka will resend the message.
- The message is replayed from the last committed offset.
- That's why offsets are committed after successful processing.
- This gaurantees:
 * No data loss
 * Automatic retries
 * Offset management in real event-driven pipelines.
- Kafka doesn't know either our business logic succeeded. It only tracks offsets.
- If processing fails -> offset is not committed -> message is retried.

2. **Best Practices**
* Read message
* Process Message
* Save data
* Commit offset

3. **Skipping bad messages (moving offset forward) - Handling corrupted or invalid data**
* Send message to a DLQ(Dead Letter Queue)
* Commit offset manually
* Continue processing next messages. This prevents the pipeline from getting stuck.

## Why product companies use Kafka for event-driven architecture:
1. Fault tolerance
2. Retryability
3. Gauranteed delivery (at-least-once + Idempotency)
4. Horizontal Scalability
5. Backpressure Handling (if consumer service database writes are slower than producing events -> using bounded queue and batch processing can add backpressure).
   
## Scenario based Questions:
Q. **Traffic Spike: How do you handle high traffic?**
sol. Load Balancing, Kafka for decoupling, and Async APIs to prevent thread blocking.

Q. **How do you design for the future?**
sol. Build Stateless Services and use event-driven messaging. 

Q. **Exponential BackOff: A Reliable Retry Strategy for Resilient Systems?**
* Exponential back off is a retry mechanism used in distributed systems widely, where the delay between retry attempts increases exponentially after each failure.
* Instead of retrying immediately or at constant interval, the system waits progressively longer before each retry. This approach prevents overwhelming failing services and improves sytem reliability.
* Exponential backoff is widely used in micorservices, APIs, cloud platforms, message queues, and n/w communications especially when dealing with transient failures like timeouts or temporary service unavailable.

* **Problems without Exponential backoff:**
* Retry storms
* Increased latency
* Cascading failure
* Server Overload

* **Benefits of exponential backoff:**
* Reduces load on failing services
* Gives sytem time to recover.
* Improves fault tolerance.
* Encourages graceful degradation

* **How Exponential Backoff grows:**
* Retry grows exponentially after each failure:
* delay = baseDelay * (2 ^ retry attempt) i.e 1s, 2s, 4s, 8s, 16s...
* often randomness is added to avoid synchronised reties across multiple clients.

```

```
