# DTS DevOps Technical Test

## Objective

To assess your ability to implement a small feature against a real database, then design and build a production-grade CI/CD pipeline around it.

**Estimated time: 2-3 hours.**

---

## Scenario

HMCTS requires a new system so caseworkers can keep track of their tasks. You will extend a provided Java backend to persist tasks to a database, then build the delivery pipeline and operational infrastructure needed to ship and run it reliably.

---

## Part 1 — Backend Feature

Fork the following repository and use it as your starting point:

**[https://github.com/hmcts/hmcts-dev-test-backend](https://github.com/hmcts/hmcts-dev-test-backend)**

The repository contains a working Spring Boot service. You need to extend it to persist tasks to a **PostgreSQL** database.

**To verify the application builds before you start:**

```bash
./gradlew bootRun

curl http://localhost:4000/                 # welcome message
curl http://localhost:4000/get-example-case # sample case JSON
```

### What to implement

Add Spring Data JPA and a PostgreSQL driver to the Gradle build, then implement two endpoints:

| Method | Path         | Description           |
|--------|--------------|-----------------------|
| POST   | `/tasks`     | Create and persist a new task |
| GET    | `/tasks/{id}`| Retrieve a task by ID |

The `Task` entity should have the following fields:

| Field       | Type     | Notes                             |
|-------------|----------|-----------------------------------|
| id          | Long     | Auto-generated primary key        |
| title       | String   | Required                          |
| description | String   | Optional                          |
| status      | Enum     | `TODO`, `IN_PROGRESS`, `DONE`     |
| dueDate     | DateTime | ISO-8601                          |

---

## Part 2 — CI/CD Pipeline & DevOps

With a working API in place, build the delivery pipeline and operational configuration around it. Everything in this part must be runnable locally — no cloud account or paid services are required.

### 1. Containerisation

- Write a `Dockerfile` for the backend service
- Use a multi-stage build to keep the final image lean
- The container should run as a non-root user
- Include a `docker-compose.yml` that runs the application and its **PostgreSQL** database locally, passing database credentials to the app via environment variables

### 2. CI/CD Pipeline (GitHub Actions)

Design a pipeline that a team would use day-to-day. At minimum it should:

- **Build & test** — compile the application and run the test suite on every push
- **Static analysis** — the `uk.gov.hmcts.java` plugin already wires Checkstyle and OWASP dependency checking into the Gradle build; ensure these run in CI
- **Container image build** — build the Docker image with a meaningful tagging strategy (e.g. git SHA, semver, or `branch-buildnumber` — document your rationale)
- **Container image push** — optional but recommended; push to Docker Hub or GitHub Container Registry
- **Security scanning** — scan the container image for known vulnerabilities using Trivy; block the pipeline on CRITICAL severity findings (HIGH is a warning)
- **Deploy** — validate and/or apply Kubernetes manifests to a cluster

Think about how the pipeline should behave differently on feature branches versus `main`, and what gates (tests passing, no critical CVEs) should block a deployment.

**CI deploy options — pick one:**

| Option | Approach |
|--------|----------|
| A (preferred) | Spin up a `kind` cluster inside the GitHub Actions runner and deploy there |
| B | Run all checks in GitHub Actions; deploy to a local cluster manually and document the exact commands |

Either is valid. Option A shows end-to-end automation; Option B is fine if you document the manual steps clearly.

### 3. Kubernetes Manifests

Provide Kubernetes manifests to deploy the full stack to a local cluster (kind or minikube). These should include:

**Application:**
- `Deployment` with appropriate resource requests and limits
- `Service` and `Ingress` configuration
- Liveness and readiness probes (the `/actuator/health` endpoint is suitable for both)

**Database:**
- `Deployment` and `Service` for PostgreSQL
- `Secret` containing the database credentials, referenced by both the PostgreSQL and application deployments
- `ConfigMap` for non-sensitive environment configuration (e.g. database name, host)

This is where secrets management becomes real: the database password must flow from a Kubernetes `Secret` into the app as an environment variable — do not hardcode credentials anywhere.

**Ingress note:** Getting Ingress working locally requires an ingress controller. For **minikube** run `minikube addons enable ingress`; for **kind** follow the [kind ingress guide](https://kind.sigs.k8s.io/docs/user/ingress/) with `extraPortMappings`. If you run into issues, a `NodePort` Service is an acceptable substitute — just note the trade-off.

**Secrets note:** A Kubernetes `Secret` is the minimum expected. For a more complete solution, consider Sealed Secrets or SOPS so encrypted secrets can be safely committed alongside the manifests.

### 4. Observability

The application already produces structured JSON logs and exposes `/actuator/health`. Demonstrate that these are accessible in your running deployment:

- Structured logs visible via `kubectl logs`
- Readiness/liveness probes wired to `/actuator/health`
- **Bonus:** expose `/actuator/prometheus` and provide a basic Prometheus scrape config or Grafana dashboard

### 5. Documentation

Update the `README.md` in your submission covering:

- How to run the application locally with Docker Compose
- How the CI/CD pipeline works and what each stage does
- How to deploy the full stack (app + database) to Kubernetes
- Any assumptions, trade-offs, or things you would do differently with more time

---

## Technical Requirements

| Area             | Requirement                                                    |
|------------------|----------------------------------------------------------------|
| Language         | Java 21 (Gradle build, as provided)                            |
| CI/CD            | GitHub Actions                                                 |
| Containerisation | Docker                                                         |
| Orchestration    | Kubernetes — kind or minikube (local, no cloud account needed) |
| Database         | PostgreSQL                                                     |
| Registry         | Docker Hub (free tier) or GitHub Container Registry            |
| Source control   | GitHub                                                         |

---

## Submission Guidelines

1. Fork the starter repository and add all pipeline and infrastructure configuration to it
2. Ensure the repository includes all GitHub Actions workflows, Dockerfile, docker-compose, and Kubernetes manifests
3. Include a `README.md` that lets an assessor run and verify your solution locally
4. Submit the repository link with your application when complete
5. Be prepared to walk through your implementation and pipeline design at interview stage

---

## Use of AI Coding Assistants

The use of AI coding assistants is permitted. However, please ensure the submission represents your own understanding, as you will be required to explain, justify and extend your work if invited for interview.

Good luck!
