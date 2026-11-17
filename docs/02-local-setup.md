# Installing JFrog Artifactory Locally

## Overview

For this tutorial, JFrog Artifactory OSS runs locally using Docker Desktop and Docker Compose.

We use PostgreSQL as Artifactory's database.

This provides an isolated learning environment where we can explore Artifactory without requiring access to a company-managed JFrog instance.

## Technologies Used

| Technology | Purpose |
|---|---|
| Docker Desktop | Runs the containers |
| Docker Compose | Defines and manages the services |
| JFrog Artifactory OSS | Stores and manages software artifacts |
| PostgreSQL | Stores Artifactory metadata and configuration information |

The installation was tested on an Apple Silicon Mac.

## Starting the Environment

The Docker Compose configuration is stored in `docker/compose.yaml`.

Before starting the environment, create a local `.env` file containing the required `POSTGRES_PASSWORD` setting.

The `.env` file must not be committed to GitHub.

From the Docker configuration directory, run:

```bash
docker compose up -d
```

To check the containers:

```bash
docker compose ps
```

Once Artifactory has started, open:

**http://localhost:8082/ui/**

![JFrog Artifactory login page](../images/01-artifactory-login.png)

*Figure 1: The JFrog Artifactory login screen running locally.*

## Stopping the Environment

Run:

```bash
docker compose down
```

This removes the containers but preserves their named Docker volumes.

To start the environment again:

```bash
docker compose up -d
```

## Notes

This configuration is intended for local learning, not production deployment.

Artifactory stores application metadata in PostgreSQL and artifact files in persistent storage.

For this tutorial, both are managed through Docker volumes.

## macOS Configuration and Troubleshooting

### Running JFrog Artifactory on Apple Silicon

This tutorial was developed using the following environment:

| Component | Configuration |
|---|---|
| Operating system | macOS |
| Processor | Apple M4 (ARM64) |
| Memory | 24 GB RAM |
| Container runtime | Docker Desktop |
| Container management | Docker Compose |
| Database | PostgreSQL |
| Artifactory edition | JFrog Artifactory OSS |

Apple Silicon Macs, including M-series processors, use the **ARM64 architecture**.

Some Docker images are designed for Intel/AMD64 processors. Although Docker Desktop can run AMD64 images on Apple Silicon using emulation, doing so may reduce performance.

For this reason, our Docker Compose configuration avoids explicitly forcing the `linux/amd64` platform.

Docker can therefore select an ARM64 image when one is available.

### Checking the Container Architecture

To check the architecture of the running Artifactory container:

```bash
docker compose exec artifactory uname -m
```

An ARM64 container should return:

```text
aarch64
```

An AMD64 container typically returns:

```text
x86_64
```

Using the native architecture avoids the overhead associated with emulation.

### Docker Desktop Resource Configuration

JFrog Artifactory runs several internal services and requires a reasonable amount of memory.

For the learning environment, the following Docker Desktop settings were selected as starting values:

| Resource | Configuration |
|---|---|
| Memory | 10 GB |
| CPUs | 4–6 |
| Available disk space | At least 30 GB |

These are suggested settings for this particular learning environment, not universal minimum requirements.

They can be adjusted according to the resources available on your computer.

To review Docker's resource allocation, open:

**Docker Desktop → Settings → Resources**

### Issues Encountered During the Initial Installation

Before switching to Docker Compose with PostgreSQL, an initial installation was attempted using a single Artifactory OSS Docker container.

Several issues were encountered.

#### Issue 1: Artifactory Would Not Start

**Symptoms**

The Docker container appeared to be running, but the JFrog web interface was unavailable.

The logs contained errors similar to:

```text
Cluster join: Failed resolving join key;
Missing join key
```

And:

```text
Master key is missing
```

**Investigation**

Although the messages suggested a problem with Artifactory's internal security keys, further examination revealed an underlying database configuration issue.

The key error was:

```text
DB Type derby is not allowed:
Cannot start the application with a database
other than PostgreSQL.
```

**Temporary solution**

For the initial installation, the following setting was enabled in Artifactory's `system.yaml` file:

```yaml
shared:
  database:
    allowNonPostgresql: true
```

This allowed Artifactory to proceed beyond the Derby database restriction.

However, Derby is not the recommended database for a new Artifactory deployment, and native ARM64 installations require an external database.

**Final approach**

The learning environment was redesigned to use PostgreSQL through Docker Compose.

This avoids relying on the embedded Derby database.

#### Issue 2: The Platform Was Warming Up

**Symptoms**

After resolving the database configuration issue, the browser displayed:

```text
The Platform is Warming Up
```

**Explanation**

Artifactory consists of multiple internal services.

During startup, these services initialise and establish communication with one another.

The warming-up message indicates that the platform has not yet completed its startup process.

**Troubleshooting**

Check the application logs:

```bash
docker compose logs --tail=50 artifactory
```

Also check that both containers are running:

```bash
docker compose ps
```

A running container does not necessarily mean that Artifactory is ready to accept requests.

#### Issue 3: Slow Login and Entitlement Errors

**Symptoms**

The JFrog login page loaded, but logging in was unusually slow.

The logs contained:

```text
First-time entitlement fetch failed:
12 UNIMPLEMENTED: Received HTTP status code 404
```

**Investigation**

The message originated from JFrog's frontend service.

A possible workaround identified during troubleshooting was to disable JFConnect in the Artifactory configuration:

```yaml
jfconnect:
  enabled: false
```

This was applied as a local troubleshooting measure.

**Important:** This is not a universal fix for slow logins. Entitlement errors should be investigated alongside the health of Artifactory's other services.

#### Issue 4: Poor Performance

**Symptoms**

Although Artifactory eventually became accessible, the web interface was slow.

Docker reported approximately:

```text
Memory usage: 5.36 GB
Memory limit: 7.75 GB
Memory utilisation: 69%
```

**Possible contributing factors**

The original container was launched with:

```bash
--platform linux/amd64
```

This explicitly requests an AMD64 container.

Because the host machine uses an Apple M4 processor, running an AMD64 container may introduce emulation overhead.

However, the architecture of the original running container was not conclusively verified, so emulation was not confirmed as the cause of the performance problem.

Artifactory also runs multiple Java-based services, making it relatively resource-intensive.

**Updated approach**

The revised installation uses:

- Docker Compose to manage the services.
- PostgreSQL instead of Derby.
- ARM64-compatible images where available.
- A larger Docker memory allocation.
- Persistent Docker volumes.

These changes are intended to improve compatibility, maintainability and performance. They do not guarantee that every performance issue will disappear.

### Useful Troubleshooting Commands

| Command | Purpose |
|---|---|
| `docker compose ps` | Check container status |
| `docker compose logs --tail=50 artifactory` | View recent Artifactory logs |
| `docker compose logs --tail=50 postgres` | View PostgreSQL logs |
| `docker stats --no-stream` | Check CPU and memory usage |
| `docker compose exec artifactory uname -m` | Check container architecture |
| `docker compose restart artifactory` | Restart Artifactory |

### Key Lessons Learned

The initial installation demonstrated several useful troubleshooting principles:

1. **A running container does not guarantee a healthy application.** Always check application readiness and logs.
2. **Error messages can be misleading.** Missing security keys were symptoms of a deeper startup problem.
3. **Database compatibility matters.** PostgreSQL is the preferred database for our ARM64 environment.
4. **Processor architecture matters.** Native ARM64 images can avoid the performance overhead of AMD64 emulation.
5. **Resource allocation matters.** Monitor CPU and memory before changing Docker settings.
6. **Use application logs to diagnose problems.** Logs often reveal the underlying cause more clearly than browser error messages.

These principles apply beyond JFrog Artifactory and are useful when troubleshooting test environments, containerised applications and CI/CD infrastructure.
