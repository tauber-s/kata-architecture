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

#### 6.1 - Class Diagram

![Class Diagram - Kata Service](images/classes/Class%20Diagram.png)

#### 6.2 - Contract Documentation

**API Specifications:**
- [PoC Service API](api-specs/poc-service.yaml) - PoC management operations
- [Kata Service API](api-specs/kata-service.yaml) - Collaborative coding sessions  
- [Reporting Service API](api-specs/reporting-service.yaml) - Analytics and reporting
- [Video Service API](api-specs/video-service.yaml) - Video compilation generation

##### Key Service Endpoints Summary

**PoC Service** (`/poc/v1`) - [Full API Spec](api-specs/poc-service.yaml):
- `POST /pocs` - Create PoC
- `GET /pocs` - Filter PoCs (supports search by keyword, language, tags)
- `GET /pocs/{id}` - Get PoC by ID
- `PUT /pocs/{id}` - Update PoC
- `DELETE /pocs/{id}` - Delete PoC

**Kata Service** (`/kata/v1`) - [Full API Spec](api-specs/kata-service.yaml):
- `POST /katas` - Create kata session
- `GET /katas/active` - List active katas
- `POST /katas/{id}/rooms` - Create room
- `PUT /katas/{id}/rooms/{roomId}/join` - Join room
- `PUT /katas/{id}/rooms/{roomId}/leave` - Leave room
- `PUT /katas/{id}/rooms/{roomId}/complete` - Complete room
- `PUT /katas/{id}/end` - End kata session

**Reporting Service** (`/reports/v1`) - [Full API Spec](api-specs/reporting-service.yaml):
- `GET /reports/usage` - Usage analytics with date range
- `GET /reports/poc-summary/{pocId}` - PoC summary report
- `GET /reports/kata-analytics` - Kata session analytics
- `GET /reports/tenant-overview` - Comprehensive tenant metrics
- `POST /reports/export` - Export reports (PDF, CSV, XLSX, JSON)

**Video Service** (`/video/v1`) - [Full API Spec](api-specs/video-service.yaml):
- `POST /videos/generate` - Generate yearly compilation
- `GET /videos` - List user videos
- `GET /videos/{id}` - Get video details
- `GET /videos/{id}/download` - Download video (pre-signed S3 URL)
- `GET /videos/{id}/thumbnail` - Get video thumbnail
- `DELETE /videos/{id}` - Delete video

## Video Service

1. Generate Yearly Video
    ```http
    POST /videos/generate/
    Request: {}
    Response:
    {
      "id": "uuid",
      "user_id": "uuid",
      "status": "processing|ready|unavailable",
      "url": "string|null"
    }
    ```

2. List User Videos
    ```http
    GET /videos/
    Response:
    {
      "id": "uuid",
      "user_id": "uuid",
      "year": 2025,
      "url": "string",
      "created_at": "..."
    }
    ```

3. Download Video
    ```http
    GET /videos/{id}/download
    Response: 302 Redirect to S3 pre-signed URL
    ```

#### 6.3 - Persistence Model

#### 6.3.1 - Tables Diagram

![Use Case - Kata Service](images/tables/Tables%20Diagram.png)

##### 6.3.2 - Main queries
1. Users by cognito_sub
   ```sql
   SELECT * FROM users WHERE tenant_id = :tenant_id AND cognito_sub = ?;
   ```

2. PoCs by filter
   ```sql
   SELECT *
   FROM pocs
   WHERE tenant_id = :tenant_id
   AND (:q IS NULL OR lower(title) LIKE '%'||lower(:q)||'%')
   AND (:language IS NULL OR :language = ANY(languages))
   AND (:tags IS NULL OR tags && :tags_array)
   ORDER BY created_at DESC
   LIMIT :limit OFFSET :offset;
   ```

3. Active Katas
   ```sql
   SELECT * FROM kata_runs
   WHERE tenant_id = :tenant_id
   AND status = 'in_progress'
   ORDER BY started_at DESC;
   ```
4. List Videos
   ```sql
   SELECT *
   FROM videos
   WHERE user_id = :user_id
   AND tenant_id = :tenant_id
   ORDER BY created_at DESC;
   ```

##### 6.3.3 - Partitioning Strategy

**Partitioning**

We considered three partitioning options for our multi-tenant database:

- **Option A – Partition by `tenant_id`:**  
  Best when we have a small number of tenants, each with large datasets. Provides strong tenant isolation.

- **Option B – Partition by time (`created_at` / `started_at`):**  
  Useful for time-based queries (recent PoCs, videos, katas).

- **Option C – Hybrid (partition by `tenant_id`, sub-partition by time):**  
  Good for many tenants each with large amounts of data. Combines isolation with time-based pruning.

**Decision:**
- At first, we will partition by `tenant_id` if we have a small number of tenants with high data volume.
- If the number of tenants grows too much, we will revert to single schema + `tenant_id` column with indexes.
- If we observe very large datasets per tenant, we adopt hybrid partitioning (`tenant_id` + `time`).

---

**Indexing**

To ensure efficient queries under our multi-tenant design, we will use the following composite indexes:

- **Users**
   - `(tenant_id, cognito_sub)` for fast lookup of users by Cognito identity.

- **PoCs**
   - `(tenant_id, created_at DESC)` for listing/filtering.
   - Index on `tags` and `languages` arrays for fast multi-value search.

- **Katas**
   - `(tenant_id, status, started_at DESC)` for querying active katas.

- **Videos**
   - `(tenant_id, user_id, created_at DESC)` for retrieving videos per user.

### 🖹 7. Migrations

IF Migrations are required describe the migrations strategy with proper diagrams, text and tradeoffs.

### 🖹 8. Testing strategy

Explain the techniques, principles, types of tests and will be performaned, and spesific details how to mock data, stress test it, spesific chaos goals and assumptions.

### 🖹 9. Observability strategy

Explain the techniques, principles,types of observability that will be used, key metrics, what would be logged and how to design proper dashboards and alerts.

### 🖹 10. Data Store Designs

For each different kind of data store i.e (Postgres, Memcached, Elasticache, S3, Neo4J etc...) describe the schemas, what would be stored there and why, main queries, expectations on performance. Diagrams are welcome but you really need some dictionaries.

### 🖹 11. Technology Stack

Describe your stack, what databases would be used, what servers, what kind of components, mobile/ui approach, general architecture components, frameworks and libs to be used or not be used and why.

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