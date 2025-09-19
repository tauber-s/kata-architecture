# 🧬 Template

This is a template created by Diego Pacheco which the goal to better describe a tought process which is called architecture. This should be used to learn software architecture and to pratice with use cases.

## 🏛️ Structure

### 1. 🎯 Problem Statement and Context

**Mr. Bill wants a multi-tenant system where:**
- Users can register their PoCs.
- Users can search PoCs by name, programming language, and tags.
- The system should allow report generation and the ability to generate a yearly video compilation of all PoCs.
- The platform must support real-time dojos (collaborative learning sessions inside the app).
- The app must be secure with proper login and tenant isolation.
- The solution must be delivered as a mobile app (native, not Ionic).
- The backend must run on AWS, not as a monolith, not with single AZ, and not serverless/Lambda.
- The database cannot be MongoDB or a single relational DB.
- It needs to be scalable, because it will be sold to many companies with multiple users.

**Context**
- **End users:** Developers.
- **Market:** Brazilian tech companies (but should be designed with global expansion in mind).
- **Business model:** SaaS (multi-tenant).

**Non-functional requirements**
- **Scalability:** support growth as adoption increases.
- **Availability:** no single AZ, so multi-AZ at minimum.
- **Security:** tenant isolation, secure authentication, encrypted storage.
- **Performance:** fast search (tags, name, language).
- **Extensibility:** future-proof for new features like AI-assisted recommendations.

### 2. 🎯 Goals

1. **Multi-tenancy:** The system must isolate data per tenant, ensuring no leakage between customers.
2. **Secure access:** Strong authentication and authorization, encrypted storage, and data-in-transit encryption.
3. **Native mobile experience:** Build the mobile app natively (Swift for iOS, Kotlin for Android).
4. **PoC management:** Create, edit, delete, and search PoCs (by name, programming language, tags).
5. **Search performance:** Search queries must return results in < 200ms for typical workloads.
6. **Reporting:** Generate report dashboards with aggregated PoC information.
7. **Video generation:** Ability to generate a yearly compilation video of all PoCs per user.
8. **Real-time dojos:** Enable synchronous code writing, chat and voice call inside the platform.
9. **Scalability & Availability:** Multi-AZ deployment on AWS, must scale horizontally.
10. **Observability:** Logging, metrics, and tracing from day one.

### 3. 🎯 Non-Goals

1. **On-premise deployments:** The system will only be SaaS, no support for local installs.
2. **Cross-cloud support:** AWS is the mandatory cloud, no support for GCP or Azure.
3. **Monolithic architecture:** The system will not be delivered as a monolith, we will design distributed services.
4. **Serverless/Lambda-first approach:** Explicitly forbidden by restriction, we’ll use containers/ECS/EKS instead.
5. **Single AZ setup:** Not acceptable, high availability is a must.
6. **NoSQL MongoDB:** Excluded by restriction, we’ll use relational and specialized stores.
7. **Single relational DB for everything:** We’ll separate concerns (e.g. PoCs in Postgres, search in Elasticsearch, media in S3).
8. **Offline-first mobile support:** Not in scope for the MVP. Users must be online.
9. **Third-party video editing features:** Beyond generating yearly compilations, full-fledged video editing is not a goal.
10. **Enterprise-level AI recommendations:** AI-based features are nice-to-have, but not in MVP scope.

### 📐 3. Principles

1. **Separation of Concerns**
    * Each service should handle a single bounded responsibility (e.g., PoC service, reporting service, video generation service).
    * This avoids accidental coupling and simplifies scaling.

2. **Multi-Tenancy by Design**
    * Tenant data must be isolated logically at the database layer (e.g., tenant_id column and row-level security) and at the storage layer (e.g., per-tenant S3 folders).
    * No shared queries without explicit tenant scoping.

3. **Security First**
    * Authentication via standards (OAuth2/OpenID Connect).
    * Authorization with role and tenant-based access.
    * Data encryption at rest (KMS, RDS encryption) and in transit (TLS 1.2+).

4. **Cloud-Native and AWS-First**
    * Use AWS managed services (RDS, ECS/EKS, S3, ElastiCache, OpenSearch) to reduce operational overhead.
    * No monoliths, no single-AZ setups, no vendor lock-in beyond AWS basics.

5. **Scalability via Horizontal Scaling**
    * All services must be stateless and run in containers.
    * Persistent state only in managed databases or object storage.

6. **Observability Built-In**
    * Collect metrics (Prometheus compatible), logs and distributed traces.
    * Dashboards and alerts defined as code (Grafana/CloudWatch).

7. **APIs as Contracts**
    * Services communicate via well-defined APIs (REST/gRPC).
    * Contracts must be versioned and backward-compatible where possible.

8. **Event-Driven for Async Work**
    * Use Kafka or AWS MSK for decoupling long-running jobs (e.g., video generation, reporting).
    * Sync APIs only for quick user-facing operations.

9. **Fail-Fast, Resilient by Default**
    * Use retries, circuit breakers, bulkheads.
    * Chaos testing baked into CI/CD pipelines.

10. **Evolvability**
    * Architecture must allow new services to be added without re-architecting.
    * ADRs (Architecture Decision Records) should capture major decisions.

### 🏗️ 4. Overall Diagrams

#### 4.1 - Overall Architecture Diagram

![Overall Arch Diagram](images/overall/Overall%20architecture.png)

#### 4.2 - Deployment Diagram

![Deployment Diagram](images/deploy/Deploy%20Diagram.png)

#### 4.3 - Use Cases Diagram

![Use Case - POC Service](images/use_cases/Use%20Case%20-%20POC%20Service.png)
![Use Case - Kata Service](images/use_cases/Use%20Case%20-%20Kata%20Service.png)

### 🧭 5. Trade-offs

List the tradeoffs analysis, comparing pros and cons for each major decision.
Before you need list all your major decisions, them run tradeoffs on than.
example:
Major Decisions: 
```
1. One mobile code base - should be (...)
2. Reusable capability and low latency backends should be (...)
3. Cache efficiency therefore should do (...)
```
Tradeoffs:
```
1. React Native vs (Flutter and Native)
2. Serverless vs Microservices
3. Redis vs Enbeded Caches
```
Each tradeoff line need to be:
```
PROS (+) 
  * Benefit: Explanation that justify why the benefit is true.
CONS (+)
  * Problem: Explanation that justify why the problem is true.
```
PS: Be careful to not confuse problem with explanation. 
<BR/>Recommended reading: http://diego-pacheco.blogspot.com/2023/07/tradeoffs.html

### 🌏 6. For each key major component

What is a majore component? A service, a lambda, a important ui, a generalized approach for all uis, a generazid approach for computing a workload, etc...
```
6.1 - Class Diagram              : classic uml diagram with attributes and methods
6.2 - Contract Documentation     : Operations, Inputs and Outputs
6.3 - Persistence Model          : Diagrams, Table structure, partiotioning, main queries.
6.4 - Algorithms/Data Structures : Spesific algos that need to be used, along size with spesific data structures.
```

Exemplos of other components: Batch jobs, Events, 3rd Party Integrations, Streaming, ML Models, ChatBots, etc... 

Recommended Reading: http://diego-pacheco.blogspot.com/2018/05/internal-system-design-forgotten.html

### 🖹 7. Migrations

IF Migrations are required describe the migrations strategy with proper diagrams, text and tradeoffs.

### 🖹 8. Testing strategy

Explain the techniques, principles, types of tests and will be performaned, and spesific details how to mock data, stress test it, spesific chaos goals and assumptions.

### 🖹 9. Observability strategy

Explain the techniques, principles,types of observability that will be used, key metrics, what would be logged and how to design proper dashboards and alerts.

### 🖹 10. Data Store Designs

For each different kind of data store i.e (Postgres, Memcached, Elasticache, S3, Neo4J etc...) describe the schemas, what would be stored there and why, main queries, expectations on performance. Diagrams are welcome but you really need some dictionaries.

### 🖹 11. Technology Stack
#### **1. General Architecture**

- **Architecture Style:** services-based architecture with REST APIs.
- **Deployment:** Containerized services orchestrated with Kubernetes
- **Scalability:** Stateless services with horizontal scaling, load balancing, and auto-scaling policies.
- **Security:**
	- **Authentication & Authorization:** Managed by **Amazon Cognito**, supporting OAuth 2.0, OpenID Connect, and JWT-based authentication.
	- **Transport Security:** All communication between services and clients is encrypted with **TLS/SSL**.
	- **Centralized Security Controls:** Fine-grained access control policies and Cognito user pools/federation for secure identity management.
	- **Observability:** Centralized logging, monitoring, and alerting to detect anomalies and potential security threats in real time.

- **Networking & Entry Points:**
	- **Amazon Route 53:** DNS management for custom domains with health checks and failover support.
	- **AWS Application Load Balancer (ALB):** Distributes traffic across microservices, terminates SSL, and integrates with WAF and Cognito.
	- **AWS WAF (Web Application Firewall):** Protects APIs from common web exploits (SQL injection, XSS, bots, DDoS).
    

---

#### **2. Backend**

- **Primary Language/Framework:**
	- TODO: add languages
- **Databases:**
    
    - **Relational Database:** PostgreSQL (primary OLTP database, ACID-compliant, strong support for JSON fields).
    - **Caching:** Redis (in-memory cache, session store, rate limiting).
    - **Search:** OpenSearch.
        
- **Message Broker:**
	- TODO: vamos ter message broker?

- **API Layer:**
    - REST (JSON-based)

---

#### **3. Frontend & UI**

- **Web:**
    - **Framework:** 
    - **State Management:**
    - **UI Library:**
    - **SSR/SSG (optional):**
        
- **Mobile:**
	- **Approach:** **Fully native implementations** to maximize performance, take advantage of platform-specific capabilities, and provide the best user experience.
	- **iOS:** Swift / SwiftUI.
	- **Android:** Kotlin / Jetpack Compose.
	- **Why Native:** Better performance, tighter integration with device hardware (camera, sensors, push notifications), and ability to adopt new platform features more quickly compared to hybrid solutions.
        

---

#### **4. DevOps & Infrastructure**

- **Hosting / Cloud:**
    - AWS — using managed services (RDS for PostgreSQL, Elasticache for Redis, etc.). 
- **Containerization:** Docker for consistent environments across development, staging, production.
- **CI/CD:**
- **Monitoring & Logging:** 
    - Prometheus + Grafana for metrics.


### 🖹 12. References

* Architecture Anti-Patterns: https://architecture-antipatterns.tech/
* EIP https://www.enterpriseintegrationpatterns.com/
* SOA Patterns https://patterns.arcitura.com/soa-patterns
* API Patterns https://microservice-api-patterns.org/
* Anti-Patterns https://sourcemaking.com/antipatterns/software-development-antipatterns
* Refactoring Patterns https://sourcemaking.com/refactoring/refactorings
* Database Refactoring Patterns https://databaserefactoring.com/
* Data Modelling Redis https://redis.com/blog/nosql-data-modeling/
* Cloud Patterns https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/introduction.html
* 12 Factors App https://12factor.net/
* Relational DB Patterns https://www.geeksforgeeks.org/design-patterns-for-relational-databases/
* Rendering Patterns https://www.patterns.dev/vanilla/rendering-patterns/
* REST API Design https://blog.stoplight.io/api-design-patterns-for-rest-web-services
