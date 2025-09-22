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

1. Kotlin/Spring Boot vs Go
```   
PROS (+)
  * Maturity & ecosystem: Spring Boot provides integrations with AWS, PostgreSQL, Redis, OpenSearch, and Kafka (MSK), accelerating delivery.
  * Robust typing & productivity: Kotlin adds concision, null-safety, and coroutines; Spring WebFlux supports reactive I/O for high-throughput REST/WebSocket endpoints.
  * Operational readiness: First-class observability (Micrometer/OpenTelemetry) and security (Spring Security + Cognito/OIDC).
CONS (-)
  * Runtime footprint: JVM baseline memory/CPU is higher than Go or Rust, increasing container cost for small instances.
  * Startup and image size: Slower startup and larger images vs Go; impacts scale-to-zero and ultra-fast rollouts.
```
2. Client side
Android and IOS </br>

Pros (+)
  * Better performance
  * Better access to native features

Cons (-)
  * Duplicated development
  * Maintenance cost
  * Risk of inconsistency between versions

WEB (NEXTJS FEBE)

Pros (+)
* SSR/SSG (server side rendering / generating)
* Client side does not know API
* Cache
* React Components

Cons (-)
* Monolith Risk
* Cache complexity



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

**Video Service** (`/video/v1`) - [Full API Spec](api-specs/video-service.yaml):
- `POST /videos/generate` - Generate yearly compilation
- `GET /videos` - List user videos
- `GET /videos/{id}` - Get video details
- `GET /videos/{id}/download` - Download video (pre-signed S3 URL)

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

No migrations, since this is a brand-new system.

### 🖹 8. Testing strategy

E2E testing: e2e can be used to ensure application key functionalities work.


### 🖹 9. Observability strategy

The observability strategy for the platform is designed to ensure **end-to-end visibility**, **early failure detection**, and **continuous security monitoring**.

#### 9.1 Techniques and Principles

We follow the principle of **Observability by Design**, with automatic collection of:

- **Metrics** (CloudWatch and Prometheus-compatible exporters): covering both infrastructure and application-level KPIs.
- **Structured logs**: including `tenant_id`, `user_id`, `request_id`, and `service`.
- **Distributed tracing**: enabled by AWS X-Ray across all microservices.
- **Threat detection**: using AWS GuardDuty and continuous audit logs via CloudTrail.
- **Dashboards and alerts as code**: managed via CloudFormation or CDK and version-controlled.

#### 9.2 Types of Observability

| Type           | Tools / Focus                                      |
|----------------|----------------------------------------------------|
| **Metrics**    | CloudWatch + Prometheus (Kubernetes, App metrics)  |
| **Logs**       | CloudWatch Logs + OpenSearch                       |
| **Tracing**    | AWS X-Ray (end-to-end distributed tracing)         |
| **Audit**      | AWS CloudTrail                                     |
| **Security**   | AWS GuardDuty                                      |

#### 9.3 Key Metrics

We track both **infrastructure performance** and **application behavior**.

##### Infrastructure (via CloudWatch):

- CPU and memory usage per pod (via Container Insights)
- Network traffic per service
- Pod restarts and health checks
- Redis (ElastiCache) latency, throughput, command rate

##### Application:

- Request latency and error rate per endpoint
- PoC creation and Kata session throughput
- Active dojo sessions (count, average duration)
- Active users per tenant
- Report generation performance

##### Security:

- Failed login attempts per IP
- Suspicious behavior and access patterns (GuardDuty)
- Audit trail for sensitive operations (CloudTrail)

#### 9.4 Logging Strategy

All services log in **structured JSON format**, with the following standard fields:

```json
{
  "timestamp": "2025-09-21T12:00:00Z",
  "level": "INFO",
  "tenant_id": "abc-123",
  "user_id": "user-456",
  "request_id": "req-789",
  "service": "kata-service",
  "message": "User joined kata room",
  "status_code": 200
}
```
Logs are sent to **CloudWatch Logs**.

Critical logs are mirrored to **Amazon OpenSearch** for advanced search and dashboarding.

---

#### 9.5 Tracing (AWS X-Ray)

Distributed tracing is enabled across all service-to-service communication and asynchronous flows.

- `request_id` and `trace_id` are propagated by default.
- Enables identification of bottlenecks, slow API calls, and high-latency services.

---

#### 9.6 Dashboard Design

Dashboards are implemented in **CloudWatch Dashboards** (optionally **Grafana** via OpenSearch), and include:

- **Platform overview**: total requests, average latency, error rate
- **Tenant dashboards**: per-tenant usage, active users, ongoing sessions
- **Service-level dashboards**: custom metrics per microservice
- **Real-time monitoring**: active dojo sessions, rooms, participants
- **Video processing**: job statuses (pending, processing, failed, completed)

All dashboards are defined as code and version-controlled through the CI/CD pipeline.

---

#### 9.7 Alerting Strategy

All critical alarms are defined using **CloudWatch Alarms** and delivered via **SNS** (email, Slack, etc.).

Alerts are based on **SLO-driven thresholds**, such as:

- HTTP error rate > 5% over 5 minutes
- API latency > 2 seconds on key endpoints
- Multiple failed login attempts within a short window
- Video generation job failures
- Spike in pod restarts in the EKS cluster

Alerts are always tested in staging before being promoted to production.

---

#### Observability Stack

| Tool           | Role                                             |
|----------------|--------------------------------------------------|
| **CloudWatch** | Metrics, logs, dashboards, alarms                |
| **AWS X-Ray**  | Distributed tracing across microservices         |
| **GuardDuty**  | Real-time threat detection                       |
| **CloudTrail** | Security auditing and AWS API usage tracking     |
| **OpenSearch** | Log indexing, advanced search, custom dashboards |


### 🖹 10. Data Store Designs

#### 10.1 Aurora (PostgreSQL)

**Purpose:**
Stores all transactional data (users, PoCs, katas, reports, metadata for videos).

**Schemas and Tables:**

##### `users`

| Column      | Type      | Definition        |
|-------------|-----------|-------------------|
| id          | UUID (PK) | Unique user ID    |
| cognito_sub | UUID (PK) | Cognito user ID   |
| name        | TEXT      | Display name      |
| email       | TEXT      | Unique, indexed   |
| created\_at | TIMESTAMP | Registration date |
| tenant\_id  | UUID      | Related Tenant    |

##### `pocs`

| Column      | Type      | Definition          |
|-------------|-----------|---------------------|
| id          | UUID (PK) | Unique PoC ID       |
| user\_id    | UUID (FK) | Owner               |
| title       | TEXT      | Title of the PoC    |
| description | TEXT      | Long description    |
| repo_url    | TEXT      | Git repository      |
| tags        | TEXT\[]   | Tags for the PoC    |
| languages   | TEXT\[]   | Code languages used |
| created\_at | TIMESTAMP | Creation date       |
| tenant\_id  | UUID      | Related Tenant      |

##### `katas`

| Column      | Type      | Definition                                    |
|-------------|-----------|-----------------------------------------------|
| id          | UUID (PK) | Kata instance                                 |
| user\_id    | UUID (FK) | Creator                                       |
| started\_at | TIMESTAMP | When kata started                             |
| ended\_at   | TIMESTAMP | When kata ended                               |
| status      | TEXT      | (`scheduled`, `active`, `failed`, `finished`) |
| tenant\_id  | UUID      | Related Tenant                                |

##### `rooms`

| Column     | Type      | Definition                       |
|------------|-----------|----------------------------------|
| id         | UUID (PK) | Room ID                          |
| kata\_id   | UUID (FK) | Related kata                     |
| name       | TEXT      | Info about challenge             |
| status     | TEXT      | (`active`, `failed`, `finished`) |
| sensei_id  | UUID (FK) | User assigned as sensei          |
| tenant\_id | UUID      | Related Tenant                   |

##### `room_participants`

| Column     | Type      | Definition                       |
|------------|-----------|----------------------------------|
| room\_id   | UUID (PK) | Room ID                          |
| user\_id   | UUID (FK) | Participant ID                   |
| joined\_at | TIMESTAMP | When participant joined the room |
| left\_at   | TIMESTAMP | When participant left the room   |
| tenant\_id | UUID      | Related Tenant                   |

##### `videos`

| Column      | Type      | Definition                                   |
|-------------|-----------|----------------------------------------------|
| id          | UUID (PK) | Video ID                                     |
| user\_id    | UUID (FK) | Owner                                        |
| year        | INT       | Year of generation                           |
| status      | TEXT      | (`pending`, `processing`, `ready`, `failed`) |
| s3\_path    | TEXT      | Pointer to S3 object                         |
| created\_at | TIMESTAMP | Request time                                 |
| tenant\_id  | UUID      | Related Tenant                               |

**Performance Expectations:**

* PoCs search optimized with indexes on tags/languages.
* Katas and rooms expected to be small (<10k rows/day).
* Reports handled by window functions + indexes.

---

#### 10.2 OpenSearch

**Purpose:**
Full-text search for PoCs and kata descriptions.

**Indices:**

* `pocs_index`

    * `id` (keyword)
    * `title` (text, analyzed)
    * `description` (text, analyzed)
    * `tags` (keyword array)
    * `languages` (keyword array)

**Main Queries:**

* `match` on description/title.
* `filter` by tags and languages.

**Performance Expectations:**

* < 1s response time for queries on < 1M PoCs.
* Updated asynchronously from Postgres via change feed.

---

#### 10.3 S3 storage

**Purpose:**
Storage for generated yearly videos.

**Buckets & Objects:**

* `{tenantId}/videos/{userId}/{year}/{videoId}.mp4`

**Metadata stored in Postgres `videos` table.**

**Performance Expectations:**

* Acceptable download latency by S3.

#### 10.4 Redis

**Purpose:**

* Caching hot queries.
* Kata rooms ephemeral states (fast join/leave tracking).

**Data Structures:**

`room:{roomId}:participants`: Set of userIds
- Add user when they join
- Remove when they leave
- Fast membership check

`room:{roomId}:roles`: Hash
- Controls who is pilot and copilot.
- `pilot`: userId
- `copilot`: userId

**Performance Expectations:**

* O(1) joins/leaves.
* Expire room keys after kata ends.

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
  - **Amazon CloudFront (CDN):** Serves static assets (frontend builds, images, documents) globally with low latency and integrates with WAF for edge security.
	- **AWS Application Load Balancer (ALB):** Distributes traffic across microservices, terminates SSL, and integrates with WAF and Cognito.
	- **AWS WAF (Web Application Firewall):** Protects APIs from common web exploits (SQL injection, XSS, bots, DDoS).
    

---

#### **2. Backend**

- **Primary Language/Framework:**
	- Kotlin (JDK 25 LTS) + Spring Boot 3.x (WebFlux, Spring Security, Data/JPA, OpenAPI) for core microservices.
- **Databases:**
    
    - **Relational Database:** PostgreSQL (primary OLTP database, ACID-compliant, strong support for JSON fields).
    - **Caching:** Redis (in-memory cache, session store, rate limiting).
    - **Search:** OpenSearch.


- **API Layer:**
    - REST (JSON-based)
    - Web Socket

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

#### **2. Deployment Pipeline**

- **CI/CD Strategy:**
  - The project uses a fully automated AWS-native CI/CD pipeline to ensure reliable, repeatable, and fast deployments.

- **AWS CodeCommit:**
  - Git-based source control repository hosting all application code and infrastructure as code (IaC).

- **AWS CodeBuild:**
  - Builds and tests container images, runs unit/integration tests, performs security scanning, and pushes images to Amazon ECR.

- **AWS CodePipeline:**
  - Orchestrates the entire CI/CD flow, triggering builds on commit or pull request merges, and promotes artifacts from dev → staging → production environments.

- **AWS CodeDeploy:**
  - Deploys updated container images to Amazon EKS using rolling updates or blue/green deployments with zero downtime.
  - Integrates with ALB target groups for automatic traffic shifting and health checks before finalizing deployment.


This pipeline ensures continuous integration, continuous delivery, and fast rollback in case of failures.

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
