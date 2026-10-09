# simple-crm-lite

A simplified Spring Boot REST API for managing customers, built to practice containerizing an application with Docker and Docker Compose.

## Requirements

- Java 21
- Maven (or use the included `./mvnw` wrapper)

## Build

```bash
./mvnw clean package
```

## Run

```bash
./mvnw spring-boot:run
```

**Note:** The app reads its database connection settings (`SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`) from environment variables, which are not set yet. Running the app will fail until these are provided — this is configured via Docker Compose in a later part of this project.
