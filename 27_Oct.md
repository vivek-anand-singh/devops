# 27th October Notes

## Syllabus

## Marks Distribution

---

## Microservice Architecture

### What is Monolithic?

Large, single codebase and deployment unit.

### Microservices

- **DNS** — Domain name resolution for services.

### Load Balancer

- **Path-based routing** — Route by URL path.
- **Instance identification** — Identify instances and distribute traffic across them.

### Authentication / Authorization

- **Authentication:** Are you a valid user?
- **Authorization:** Are you authorized to do the task?
- There can be multiple instances of these services.

### Scalability

- **3 axes of scalability** (see book reference).
- Each microservice should have its own database (database per service).
- Shared/cache data (e.g. sessions, tokens) is stored in databases like **Redis** (cache service).

### Account Service

(Service responsible for user/account operations.)

### Polyglot

**Polyglot DevOps** refers to DevOps environments or engineers who work with multiple programming languages and technologies across the software delivery pipeline.

### Load Balancer and Service Discovery

- If there are multiple instances of microservices, a load balancer should sit in front of them.
- Every load balancer should use **service discovery**: the LB must know when a new instance comes up or an existing one goes down.

---

## Microservice to Microservice Communication

### Synchronous

- Services talk directly via HTTP/REST (often through a load balancer).

### Asynchronous (Eventual Consistency)

- Use message brokers: **RabbitMQ**, **Kafka**, etc.

**Methods of asynchronous communication:**

1. **Publisher–Subscriber (Pub/Sub)** — Publisher publishes a message; subscribers are notified and consume it. Message can be consumed by multiple subscribers.
2. **Queue** — One of the instances consumes the message and then that message is removed (single consumer per message).

---

## Observability

### Event / Metrics Syncing

- Track: How long did a call take? Which instance was called?

### Logging

- **Logs** — Record system issues and events.
- For microservices, logs are typically sent to an **external logging service**.
- **Tools:** Splunk, ELK (Elasticsearch, Logstash, Kibana), DataDog.

---

## The Twelve-Factor App

A set of 12 best practices for building scalable, maintainable, and portable software-as-a-service (SaaS) applications.

- **Reference:** https://12factor.net/

---

## API Gateway

Central entry point for client requests; handles routing, auth, rate limiting, etc.

---

## Dependencies

- **Service dependency** — One microservice depends on another.
- **Data dependency** — Shared or replicated data.
- **Component dependency** — Using an SDK or shared library.
