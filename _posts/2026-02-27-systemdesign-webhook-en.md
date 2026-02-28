---
title: "System Design - Scalable & Reliable Webhook System (EN)"
layout: single
Typora-root-url: ../
categories: SystemDesign
tag: [python, fastapi, mysql, redis, celery, docker, system-design]
use_math: true
---

## Scalable & Reliable Webhook System Design

Tech Stack: `FastAPI` `MySQL` `Redis` `Celery` `Docker`

Beyond simply receiving POST requests, this system is designed to guarantee both **real-time delivery** and **reliability** simultaneously. The goal is an architecture capable of handling tens of thousands of event spikes per second while ensuring zero data loss even in the face of network failures.

---

## Background

Webhooks allow systems to send real-time notifications triggered by specific events. Unlike traditional APIs that rely on **polling**, webhooks **push** data immediately when an event occurs, maximizing both efficiency and real-time responsiveness.

![Traditional API vs Webhook]({{site.url}}/images/2026-02-27-systemdesign-webhook/traditional-vs-webhook.png){: .align-center}

| Approach | How It Works | Characteristics |
|---|---|---|
| **Polling** | Repeatedly asking "Is the food ready?" | Inefficient, wastes network resources |
| **Webhook** | Ringing the bell — "Your food is ready!" | Event-driven, real-time, up to 90% resource savings |

Real-world examples:
- **Stripe**: Payment completed, refund issued
- **Shopify**: Order created, shipment update
- **GitHub**: PR merged, issue opened

---

## 1. Functional Requirements

Extract the **verbs** from the problem description to define core features.

| Action | Feature | API |
|---|---|---|
| **Ingest** | Receive event notifications (JSON) from external services | `POST /webhook` |
| **Verify** | Authenticate requests via HMAC signature | Middleware |
| **Process** | Execute business logic (payment approval, authorization, etc.) | Async worker |
| **Monitor** | Query processed event status and logs | `GET /api/events` |

### Out of Scope

- Webhook subscription registration/management (Provider side)
- Per-event-type business logic details
- UI dashboard

---

## 2. Non-Functional Requirements

Extract the **adjectives** from the problem description to define quality constraints.

| Constraint | Description |
|---|---|
| **Low Latency** | Return 200 OK within 200ms of receiving a request |
| **Idempotency** | Prevent duplicate processing even if the same event is sent multiple times |
| **At-least-once delivery** | Guarantee at least one processing attempt with no data loss |
| **High Availability** | Service continues even when system components fail |
| **Load Buffering** | Message queue absorbs traffic spikes (back-pressure) |

### Scale Estimation

- **Daily event volume**: 1 million (5x spike during peak hours)
- **Latency target**: End-to-end under 200ms
- **Data retention**: 30 days for audit purposes
- **Event size**: ~5KB → ~150GB total storage for 30 days

---

## 3. Data Model

Extract the **nouns** from the problem description to define entities. State management based on `event_id` is essential to prevent duplicate processing.

| Table | Role | Key Attributes |
|---|---|---|
| **ProcessedEvents** | Idempotency check and result storage | `event_id`(PK), `payload`, `status`(SUCCESS/FAIL), `created_at` |
| **RetryLogs** | Retry history for failed events | `id`(PK), `event_id`(FK), `retry_count`, `last_error`, `next_retry_at` |

A **Unique Index** on `event_id` prevents duplicate inserts at the DB level.

---

## 4. Core Architecture: Decoupling Ingestion from Processing

The heart of webhook design is: **"Accept fast, process later."**

### Limitations of the Basic Design

The naive design has the Request Handler performing both HTTP reception and business logic simultaneously.

![Webhook Basic Design]({{site.url}}/images/2026-02-27-systemdesign-webhook/basic-design.png){: .align-center}

**Problem**: If the handler fails after processing but before persisting, the event is lost. During traffic spikes, DB connection pools can be exhausted, taking down the server.

### Improved Design with Message Queue

![Webhook High-Level Design]({{site.url}}/images/2026-02-27-systemdesign-webhook/high-level-design.png){: .align-center}

**Ingestion Path**:
1. Client (Provider) sends `POST /webhook`
2. FastAPI validates HMAC signature, then immediately enqueues to **Redis (Message Queue)**
3. Returns **200 OK** to the client instantly (minimizes latency)

**Processing Path**:
1. Celery Worker dequeues events one by one
2. MySQL Check: verify `event_id` hasn't been processed (idempotency guard)
3. Business Logic: execute task and persist result to DB
4. Retry Logic: on failure, schedule retry with Exponential Backoff

![Webhook Sequence Diagram]({{site.url}}/images/2026-02-27-systemdesign-webhook/sequence-diagram.png){: .align-center}

This design simultaneously provides **Failure Recovery**, **Load Buffering**, and **Horizontal Scalability**.

---

## 5. Scaling & Reliability Strategy

### Per-Component Failure Handling

**1. Request Handler Failures**

![Request Handler Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/request-handler-failure.png){: .align-center}

| Scenario | Strategy |
|---|---|
| Fails before enqueue | Client doesn't receive 200 OK → retries automatically |
| Timeout | Circuit Breaker cuts cascading failures, immediately signals failure to client |

**2. Message Queue Failures**

![Message Queue Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/message-queue-failure.png){: .align-center}

| Strategy | Description |
|---|---|
| **Durable Queue** | Persist messages to disk — survive server restarts |
| **Multi-Node Replication** | Replicate across Kafka/RabbitMQ cluster, tolerating single-node failure |

**3. Queue Consumer Failures**

![Consumer Failure]({{site.url}}/images/2026-02-27-systemdesign-webhook/consumer-failure.png){: .align-center}

| Strategy | Description |
|---|---|
| **Multiple Instances** | If one consumer fails, another picks up the workload |
| **Message Acknowledgment** | Only dequeue after DB write succeeds — failed events stay in queue |
| **Auto-Restart (Kubernetes)** | Automatically restarts failed consumers, scales based on queue depth |

**4. Database Failures**

| Strategy | Description |
|---|---|
| **Exponential Backoff** | Retry writes at $2^n$ second intervals (prevents DB overload) |
| **Replication & Failover** | Automatic failover from primary to secondary on failure |

### Exponential Backoff

Retrying immediately on external server failure only adds more load. Increasing retry intervals by $2^n$ seconds gives the system time to recover.

$$\text{Retry Interval} = 2^n \text{ seconds}, \quad n = 1, 2, 3, ...$$

---

## 6. Deep Dive: Engineering Insights

### Security: HMAC Signature Verification

Since webhooks originate externally, **source authentication** is non-negotiable.

**HMAC Signature Flow**:
1. The Provider (e.g., Stripe) and the service share a **Secret Key** in advance
2. Provider generates an HMAC hash of the payload and includes it in request headers
3. Service recalculates the hash using the same secret and validates the match

```python
import hmac
import hashlib

def verify_signature(payload: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(), payload, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

Additional security layers:

| Technique | Description | Effect |
|---|---|---|
| **IP Whitelisting** | Accept requests only from known Provider IPs | Blocks unauthorized IPs |
| **Rate Limiting** | Cap maximum requests per minute | Defends against DoS attacks |
| **HTTPS Only** | Transport layer encryption | Prevents packet interception |

### Handling Duplicate Requests (Idempotency)

Network retries can deliver the same event multiple times.

**Idempotency Key Pattern**:

```sql
-- Unique Index on event_id blocks duplicate inserts at the DB level
INSERT INTO ProcessedEvents (event_id, payload, status)
VALUES (:event_id, :payload, 'PROCESSING')
ON DUPLICATE KEY IGNORE;
```

Before processing, check if the `event_id` already exists. If it does, skip immediately to prevent duplicate execution of business logic.

### Handling Out-of-Order Events

Network delays can cause `invoice.paid` to arrive before `invoice.created`.

**Core Principle**: Webhook processing logic must **never assume event ordering**.

```
On receiving invoice.paid:
  1. Fetch latest invoice data directly from Stripe API (Source of Truth)
  2. Update local DB → set status to PAID

On receiving invoice.created (arriving late):
  1. Compare event's created_time with DB's updated_time
  2. If DB is already up-to-date → skip (prevents stale overwrite)
```

**Design Principles Summary**:

| Principle | Implementation |
|---|---|
| **Stateless processing** | Use Source of Truth (Provider API), not local DB state |
| **Idempotent design** | Processing the same event N times must yield the same result |
| **Timestamp-based skip** | Detect and skip outdated events using `event_id` + `timestamp` |

---

## 7. Summary & Retrospective

### Level Expectations

| Dimension | Junior | Senior | Staff |
|---|---|---|---|
| **Reliability** | Simple API reception | Retry and idempotency design | DLQ and failure propagation isolation |
| **Performance** | Synchronous processing | Async design with message queue | Kafka partitioning strategy |
| **Security** | No authentication | HMAC signature + IP whitelist | mTLS and payload encryption |
| **Order handling** | Assumes ordered events | Timestamp-based skip | Event sourcing and CQRS patterns |

### Key Takeaways

System design isn't just about "code that works" — it's about **"designing for every possible failure."**

The essence of a webhook system comes down to three principles:

1. **Accept fast, process later** — Decouple to minimize response latency
2. **Assume failure** — Every component can fail. Compensate with retries and idempotency
3. **Never trust ordering** — Networks don't guarantee order. Always verify against the Source of Truth
