# Microservices

- small **autonomous** services that **work together**, modelled around a **business domain**
- autonomous = indepdently releasable
- in microservices you have more choices to make - every service can have its own technology stack, persistence layer, idiomatic design; in monolithic apps, that's not the case (a lot less decisions to make)

- Principles of Microservices:

1. Modelled around business domain - more stable APIs
2. culture of automation - we have much more deployable units
3. hide implementation details - to allow one service to evolve independently of another
4. decentralize all the things -
5. deploy independently
6. consumer first
7. isolate failure
8. highly observable

### Modelling around a business domain

- do not services horizontally - having the same model of the application layering (presentation, business logic and data store)
- if some change is required that usually means cutting a way through these layers and services which are owned by teams, which requires a coordination
- having vertical boundaries is more about the business domain, the actual functionalities
- how FE fits into the picture - having a scaffolding service to collect the web ui pages or BFF to act as a single edge service for the mobile UI
  (BFF is tighly coupled to that frontend then)

### Culture of automation

- infrastructure automation - having dev/test envs automated, production platform management etc.
- automated testing
- continuous delivery - is every concrete change treated as a release candidate

### Hide implementation details

- image a situation where two service share the same database
- what if we want to change the name of a column -> one service is OK with that, but the other one is unfairly hit - we exposed the internal implementation with this. A bad thing to do. Hide the databases behind the API.
- what to reveal from your API - think about the context, internal and external with respect to domain of the service. Some service from an internal domain could require much more data than the external one. When you have some data hidden, it's easier to add them, than the opposite (if some data is there, to remove them)
- essential to keep the implemention details private to allow the seamless evolution of a service

### Decentralize all the things

- e.g. some message brokers got to smart - they know about domains, tend to solve the data consistency problems
- the principle to follow: dumb pipes, smart endpoints

### Deploy independently

- if you have a set of 5 service that must be deployed together, fix that before you get the sixth one
- how many services per host? Host = an isolated environment with its own operating system - a physical machine, a virtual machine, a container
- one service per host vs multiple services per host
- problems with multiple services per host:
  - one of the services is eating the CPU, other services are affected
  - one of the services has a prerequisite that has to be installed on that machine, but that might clash with other services dependencies
- the simpler and more scalable and manageable option -> one service per host

- consumer driven contract: if I change one service, will its consumer crash
  - test it in CI with consumer expectations tests -> when the service is changes, the tests of its consumer expectations are run to verify do they still operate

https://github.com/orgs/pact-foundation/repositories?type=all

- co-exist endpoints; let's say we are creating a new API or changing the existing one, so consumer has to adapt. How to deploy them when they are binded?
  - introduce a v2 endpoint and deploy that. The consumer is still at v1.
  - give some time to consumer to adopt the new version and change their communication contract to work with v2
  - deploy consumers
  - if not needed anymore, decommission the old v1 endpoint
- another option is to create another version of the service
  - makes the things complicated - if we need to change something about that logic, we have to change multiple services, service discovery is more complex (different versions of the same service)

### Consumer first

- the documentation, that's it
- service discovery
- what about the HumaneRegistry - https://martinfowler.com/bliki/HumaneRegistry.html

### Isolate failure

- if the service is spanned across multiple machines, it is more prone to failure: resiliency issue, network partitioning

#### Strangler application

- how the migration from monolith to microservices or to a new isolated API can work:
  1. create a so-called strangler application that will act as a proxy
  2. all incoming requests should go over the Strangler app
  3. the Strangler app should proxy the requests, if there is a specialized services for some operation, it will send it to that service
  4. when all services are isolated, all requests are forwarded to microservices, not to the original macroservice/monolith
- a bad way to crash in microservices architecture - fail slowly, resources are locked easily, practically all threads in Stangler app can be exhausted by waiting for the problematic service to respond (and it takes too much time to respond in a failing manner). How to solve it?
  - tweak the timeouts - fail earlier
  - instead of having one thread pool, let's have a thread pool per downstream application - bulkheading pattern. If one of the services is dying, the other services will be fine.
  - circuit breakers. Like in a house where a switch opens, it stops the flow of electricity. Here the circuit breaker opens when the condition is met. So, no requests are being forwarded to a target service, giving it some time to recover, especially with exponential backoff and retries. This helps with fail fast approach. Also, if we are deploying the new version of a service, we can flip it off, so no requests are forwarded, like we do this in our house to protect us when drilling the wall.

### Highly observable

Logs:

- Good, but costly: Splunk
- Good: ELK stack
- Papertrail...

Stats (aggregation with time series):

- Graphite
- Prometheus
- New relic
- App dynamics

- Correlation id: an identifier of a distributed request that is propagated along the services that are hit when some action is executed
- suitable for tracing; Zipkin

An interesting tool: Chaos monkey
Microservice should be small enough to fit into my head. 😄

Monolith vs microservices:

Monolith:

- simplicity
- consistency
- inter-module refactoring

Microservices:

- partial deployment
- availability
- preserve modularity
- multiple platforms
