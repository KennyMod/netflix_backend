# netflix_backend

Spring Boot REST API for a movie catalogue, backed by MongoDB Atlas.

Part of a two-repo deployment project: [netflix_frontend](https://github.com/KennyMod/netflix_frontend) provides the UI.

## Stack

- Java 17, Spring Boot 3.0.1
- Spring Data MongoDB
- Maven
- Docker (multi-stage build), AWS ECR, GitHub Actions

## API

| Method | Path | Returns |
|---|---|---|
| GET | `/api/v1/movies` | All movies |
| GET | `/api/v1/movies/{imdbId}` | One movie by IMDB ID |
| POST | `/api/v1/reviews` | Creates a review from `{reviewBody, imdbId}` |

## Configuration

Credentials are never committed. `application.properties` contains no secrets —
the MongoDB connection string is supplied at runtime.

**Local development.** Copy the template and fill in your own Atlas connection string:

```bash
cp .env.example .env
set -a; source .env; set +a
```

`.env` is gitignored.

**Deployment.** The connection string is passed as the `SPRING_DATA_MONGODB_URI`
environment variable, which Spring Boot binds directly to `spring.data.mongodb.uri`.
This bypasses property-placeholder resolution and is the mechanism used in the
Docker Compose stack.

## Run locally

Requires **Java 17**. Newer JDKs are not supported — the Lombok version pinned by
Spring Boot 3.0.1 fails silently on Java 21+, producing a successful build whose
model classes have no getters, and an API that returns empty JSON objects.

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 17)   # macOS
./mvnw clean package -DskipTests
java -jar target/*.jar
```

The API listens on port 8080.

## Run in Docker

```bash
docker build -t movie-backend:local .
docker run --rm -p 8080:8080 --env-file .env movie-backend:local
```

The image is a multi-stage build: Maven and the JDK stay in the build stage, and
the runtime stage carries only a JRE and the jar, running as a non-root user.
Final size is roughly 371 MB, down from about 800 MB for the single-stage
original (kept as `Dockerfile.original` for comparison).

## Status

Project complete. The AWS infrastructure (EC2, ECR, IAM) was decommissioned after
assessment, so the CI/CD workflows no longer run to completion. The workflow
definitions in `.github/workflows/` remain as a record of the Build → Push → Deploy
pipeline that was in use.
