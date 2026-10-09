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

**Note:** The app reads its database connection settings (`SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`) from environment variables, which are not set yet. Running the app will fail until these are provided — this is configured via Docker Compose (see below), which is how this app is meant to run.

## Docker

Build the image:

```bash
docker build -t simple-crm-lite:latest .
```

Verify it was created:

```bash
docker images | grep simple-crm-lite
```

**Note:** Don't run this image on its own with `docker run` — it needs a database and the `SPRING_DATASOURCE_*` environment variables. Docker Compose provides both (see below).

## Run with Docker Compose

Start the app and database together:

```bash
docker compose up -d --build
```

Watch the application logs:

```bash
docker compose logs -f app
```

Check both containers are running:

```bash
docker compose ps
```

The app is then reachable at [http://localhost:8080](http://localhost:8080).

## Testing the API

```bash
# List all customers
curl http://localhost:8080/customers

# Create a customer
curl -X POST http://localhost:8080/customers \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Tony","lastName":"Stark","email":"tony@starkindustries.com","contactNo":"55566677","jobTitle":"CEO","yearOfBirth":1980}'

# Read one customer (replace {id} with the id from the create response)
curl http://localhost:8080/customers/{id}

# Update a customer
curl -X PUT http://localhost:8080/customers/{id} \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Tony","lastName":"Stark","email":"ironman@starkindustries.com","contactNo":"55566677","jobTitle":"Superhero CEO","yearOfBirth":1980}'

# Delete a customer
curl -X DELETE http://localhost:8080/customers/{id}
```
