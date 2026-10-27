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

## API Tests (Bruno)

The `api-tests/` folder is a [Bruno](https://www.usebruno.com/) collection that runs a full CRUD cycle and checks every response. Each request checks the status code, and most also check the response body. "Add a new customer" saves the new customer's id, so the requests that follow work on that customer - not on a fixed id.

| # | Request | Expected |
|---|---|---|
| 1 | Get all customers | `200`, a list of customers |
| 2 | Add a new customer (John Doe) | `201`, saves the new `id` |
| 3 | Get customer by id | `200`, returns John Doe |
| 4 | Update customer by id (to Tony Stark) | `200`, returns the updated customer |
| 5 | Delete customer by id | `204 No Content` |
| 6 | Get deleted customer | `404 Not Found` |
| 7 | Add another new customer (Natasha Romanoff) | `201` - stays in the database, e.g. for the data persistence check below |

### Run the tests

Install the [Bruno CLI](https://docs.usebruno.com/bru-cli/overview) (`npm install -g @usebruno/cli`), start the app (`docker compose up -d --build`), then run the collection from **inside** the `api-tests` folder - the CLI treats the current folder as the collection:

```bash
cd api-tests
bru run
```

`bru run` sends every request in order and prints a pass/fail summary. In the Bruno app: *Open Collection* -> select the `api-tests` folder -> right-click the collection -> *Run*.

### Local and prod environments

The requests use one variable, `baseUrl`. With no environment it defaults to `http://localhost:8080` (Docker Compose), set in `api-tests/opencollection.yml`. Two environments in `api-tests/environments/` override it:

| Environment | File | `baseUrl` |
|---|---|---|
| `local` | `environments/local.yml` | `http://localhost:8080` (Docker Compose) |
| `prod` | `environments/prod.yml` | The live service on Render |

```bash
bru run --env local   # test the app running in Docker Compose
bru run --env prod    # test the live deployment
```

In the Bruno app, pick the environment from the dropdown in the top-right corner before running the collection.

To test another deployment without editing a file, override the variable on the command line:

```bash
bru run --env-var baseUrl=https://simple-crm-lite-production.up.railway.app
```

> Each run adds Natasha Romanoff again, so the list of customers grows over time. That's expected - the collection only deletes the customer it created in step 2.

## Data Persistence

PostgreSQL data lives in the named volume `postgres-data`, not in the container, so it survives container removal:

```bash
# Removes containers and network, but keeps the volume (data is safe)
docker compose down

# Restart - same data is still there
docker compose up -d

# Query the database directly to confirm
docker compose exec db psql -U postgres -d simplecrmlite \
  -c "SELECT id, first_name, last_name FROM customer ORDER BY id;"
```

**Warning:** `docker compose down -v` also deletes the volume, permanently erasing the data. Never use `-v` on data you need to keep.
