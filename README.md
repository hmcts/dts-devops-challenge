# DTS DevOps Technical Test

## Objective

To assess your ability to containerise a working application, build a production-grade CI/CD pipeline around it, and define its cloud infrastructure with Terraform.

**Estimated time: ~3 hours.**

---

## Scenario

HMCTS requires a new system so caseworkers can keep track of their cases. You are given a working Spring Boot service that is pre-wired for PostgreSQL. Your job is to connect it to its database and build the delivery pipeline and infrastructure definition needed to ship and run it reliably.

---

## Getting Started

Fork the following repository and use it as your starting point:

**[https://github.com/hmcts/hmcts-dev-test-backend](https://github.com/hmcts/hmcts-dev-test-backend)**

The repository contains a working Spring Boot service. **You do not need to write any Java** — this test is about configuration, pipelines, and infrastructure, not application code. To verify it builds and runs:

```bash
./gradlew build
./gradlew bootRun

curl http://localhost:4000/                  # welcome message
curl http://localhost:4000/get-example-case  # sample case JSON
```

---

## Part 1 — Database Wiring & Containerisation

The service is stateless as shipped, but it is pre-wired for **PostgreSQL**: `application.yaml` contains a commented-out, environment-variable-driven datasource section, and the comments explain how to enable a database-backed health check. Configuration changes only — no new endpoints or Java code are required.

**Database wiring:**

- Uncomment the `datasource` section in `application.yaml` and add the PostgreSQL driver and `spring-boot-starter-jdbc` (or `spring-boot-starter-data-jpa`) dependencies to `build.gradle`
- Expose the `health` actuator endpoint (the exposure list currently only includes `info`) and enable the `db` readiness group, so `curl http://localhost:4000/health` proves database connectivity

**Containerisation:**

- Write a `Dockerfile` for the backend service
- Use a multi-stage build to keep the final image lean
- The container should run as a non-root user
- Include a `docker-compose.yml` that runs the application and its PostgreSQL database locally, passing database credentials to the app via environment variables — do not hardcode credentials anywhere

**You do not need to install PostgreSQL** — the docker-compose stack is the expected way to provide it, using the official `postgres` image. Note that once the datasource is enabled, the app needs a reachable database to start; if you want to test before your compose file is ready, a one-off container will do:

```bash
docker run -d -e POSTGRES_PASSWORD=localdev -e POSTGRES_DB=devtest -p 5432:5432 postgres:16
```

**Bonus:** add container healthchecks to the compose file, wired to the `/health` endpoint.

---

## Part 2 — CI/CD Pipeline

Design a GitHub Actions pipeline that a team would use day-to-day. At minimum it should:

- **Build & test** — compile the application and run the test suite on every push
- **Static analysis** — the `uk.gov.hmcts.java` plugin already wires Checkstyle into the Gradle build; ensure it runs in CI
- **Container image build** — build the Docker image with a meaningful tagging strategy (e.g. git SHA, semver, or `branch-buildnumber` — document your rationale)
- **Image security scanning** — scan the container image for known vulnerabilities using a scanner such as Trivy; block the pipeline on CRITICAL severity findings (HIGH is a warning)
- **Terraform checks** — run `terraform fmt -check` and `terraform validate` against your infrastructure code (see Part 3)

Think about how the pipeline should behave differently on feature branches versus `main`, and what gates (tests passing, no critical CVEs, valid Terraform) should block a merge or release.

---

## Part 3 — Infrastructure as Code: Terraform on Azure

Write a Terraform configuration that defines the production infrastructure for this service on **Azure**. This is a **design and validate** exercise — **you will not apply it and you do not need an Azure account.**

### What to define

- A resource group for the service
- **Azure Database for PostgreSQL Flexible Server**, plus a database for the application
- A compute service to run the container image — **Azure Container Apps** or **App Service for Containers**, your choice (briefly justify it)
- **Azure Key Vault** holding the database credentials, with the application configured to consume them — the password must never appear in plain text in the repository
- Application configuration (database host, name, port) passed to the container as environment variables

### What we're looking for

- Sensible project structure: `variables.tf`, `outputs.tf`, type constraints and descriptions on variables, consistent naming and tagging
- No secrets committed to the repository — sensitive values should be variables marked `sensitive`, sourced from Key Vault or pipeline secrets
- A short note in your README on how state would be managed in a real deployment (e.g. `azurerm` backend with a storage account); the backend block itself may be commented out so `terraform validate` runs locally without credentials

### Verification

`terraform fmt -check` and `terraform validate` must pass locally and in your pipeline — neither requires an Azure account. Note that `terraform plan` against the `azurerm` provider **does** require authentication, so it is not expected.

---

## Part 4 — Documentation

Update the `README.md` in your submission covering:

- How to run the application locally with Docker Compose
- How the CI/CD pipeline works and what each stage does
- An overview of the Terraform configuration and how it would be deployed in a real environment
- Any assumptions, trade-offs, or things you would do differently with more time

---

## Submission Guidelines

1. Fork the starter repository and add all pipeline and infrastructure configuration to it
2. Ensure the repository includes all GitHub Actions workflows, the Dockerfile, docker-compose file, and Terraform configuration
3. Include a `README.md` that lets an assessor run and verify your solution locally
4. Submit the repository link with your application when complete
5. Be prepared to walk through your implementation, pipeline design, and infrastructure choices at interview stage. There may also be an extension exercise at this point.

---

## Use of AI Coding Assistants

The use of AI coding assistants is permitted. However, please ensure the submission represents your own understanding, as you will be required to explain, justify and extend your work if invited for interview.

Good luck!
