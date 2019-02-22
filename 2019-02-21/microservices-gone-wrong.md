# Microservices Gone Wrong

Anthony Ferrara @ircmaxwell

[slides](https://docs.google.com/presentation/d/1Ogejf47b7k0RWU-7lE_uBsCmJxL3CvydOl8fVukuQ_4/)

---

Looked at the processes involved in rebuilding a product from a monolith to a a collection of microservices

## Background

- Legacy system based on multiple frameworks and packages, multiple layers of complexity (1000+ cron jobs!)
- Limited experience working with microservices
- Product was in feature freeze for 9 months in an attempt to refactor

Requirement: Faster, better, more scalable development needed by the business

Plan: Build a second product, migrate old to new, remove old.

## Architecture

- API gateway: acts as a boundary between public and private realms. Handling auth, rate limiting etc. They used tyk.io but AWS API gateway was also an option. The API gateway creates a stable interface for in-house and third-party applications to build against
- All functionality resides in self-contained services
- Data is shared via an event store in RabbitMQ. (Kafka would have been better as it allows deterministic ordering)
- Infrastructure: Mesos + Marathon + ELB + HAProxy = they built Kubernetes before Kubernetes
- Logging+Tracking: Logspot + StatsD + Zipkin  

### Services

- Domain Services: encapsulate business logic - communicate over REST
- Async Services: long running or batch jobs - communicate over RPC
- Meta Services: bind other services together to present a more useful interface

service.json file configures each service for automatic deployment in dev and production

But service.json build process was slow, error-prone and not universally owned so it didn't get improved and therefore didn't get used.

## Deployment

Production deployment was slow: 
- environment differnces
- idealised infra expected
- systemic delays (engineers have moved on by the time bugs surfaced)
- rapidly evolving APIs
- versioning and dependency management between services was complicated
- lack of e2e tests meant parts worked individually but not as a system

## Making Changes

Change was hard:
- dependencies between services meant multiple versions for internal calls had to be accounted for

## Lessons

- Don't build microservices unless you have tooling and team to manage them
- Service calls are many magnitudes less reliable than direct code calls (how often does a PHP method not get called for some reason? Basically never)
- Build services bigger than you think and break apart if needed: 'microliths'
- Live with failure. Include it as a natural part of development
- Define service level objectives early. What are the business objectives of each service? How can we measure them?
- 'Conservation of complexity': Complexity is everywhere always. Recognise it, manage it, don't ignore it.  
