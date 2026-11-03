
# Day 33/90 — Docker Compose Multi-Service Lab

**Challenge:** DevOps Winter Arc 
**Day:** 33
**Environment:** WSL Ubuntu  
**Topics:** Docker Compose, Multi-Service Application, App + Database, Networks, Volumes, Environment Variables, Health Checks, Troubleshooting

---

## 1. Lab Overview

The objective of Day 33 is to define and run the Day 30 integration-project application using Docker Compose.

This lab focuses on running multiple services together, connecting the application to a supporting service, verifying communication, and testing persistent data.

The main tasks include:

- Creating a `compose.yaml` file.
- Defining application and database services.
- Configuring a shared internal network.
- Connecting services using Compose service names.
- Configuring environment variables.
- Keeping local credentials in an untracked `.env` file.
- Creating a safe `.env.example` file.
- Configuring database health checks.
- Starting and inspecting the Compose stack.
- Verifying database persistence.
- Performing a controlled break/fix exercise.

**Important:** Continue using the same application and repository from Days 30–32. If the application is a static website, run Nginx and PostgreSQL separately and clearly state whether the website actually connects to the database.

---

## 2. Environment Preparation

### Step 1: Open WSL Ubuntu

Check your Linux environment:

```bash
whoami
pwd
uname -a
```

### Step 2: Check Docker and Docker Compose

```bash
docker --version
docker compose version
docker info
```

These commands check:

- Docker CLI version.
- Docker Compose availability.
- Docker engine status.

### Step 3: Navigate to the Day 30 project

```bash
cd ~/DevOps-Winter-Arc
ls -la
```

Enter the actual application directory if required:

```bash
cd /path/to/your/day30-app
pwd
ls -la
```

Check that your Day 32 Docker files exist:

```bash
ls -la Dockerfile .dockerignore
```

The Day 33 lab reuses the Dockerfile and application from Day 32.

### Step 4: Create the Docker configuration directory

From the repository root:

```bash
cd ~/DevOps-Winter-Arc
mkdir -p docker
```

---

## 3. Task A — Define Services in compose.yaml

Docker Compose allows multiple containers to run together as one application stack.

For this lab, the two services are:

| Service | Purpose |
|---|---|
| `app` | Runs the Day 30 application |
| `db` | Runs the PostgreSQL database |

### Step 5: Create `.env.example`

This file contains example configuration values without real credentials.

From the repository root:

```bash
nano .env.example
```

Add:

```env
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=replace_with_a_local_password
```

Save the file.

### Step 6: Create the actual `.env` file

```bash
nano .env
```

Add:

```env
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=your_own_local_password
```

Replace the example password with a strong local password.

Save the file.

**Security:** Never commit the real `.env` file or place credentials in the Dockerfile or image.

### Step 7: Update `.gitignore`

Open:

```bash
nano .gitignore
```

Add:

```text
.env
.env.*
!.env.example
```

This excludes actual environment files while allowing the safe `.env.example` file to be retained.

### Step 8: Create compose.yaml

The following configurations are alternatives. Use the one that matches your Day 30 application.

#### Option A: Node.js application with PostgreSQL

Use this configuration if your application supports PostgreSQL and understands the database environment variables.

From the repository root:

```bash
nano docker/compose.yaml
```

Add:

```yaml
services:
  app:
    build:
      context: ../app
    container_name: winter-arc-app
    ports:
      - "8080:3000"
    environment:
      DB_HOST: db
      DB_PORT: 5432
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    container_name: winter-arc-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db_data:
```

Important points:

- This example assumes your application is inside the `app/` directory.
- The Node.js application listens on container port `3000`.
- PostgreSQL listens on container port `5432`.
- The database is not published to the host.
- `DB_HOST: db` connects the application to the Compose service named `db`.
- `service_healthy` waits for the database health check before starting the application.

Your application must actually support PostgreSQL and these environment variables. Compose configuration alone does not create database integration inside application code.

#### Option B: Static website with a separate PostgreSQL service

Use this configuration if your Day 30 application is a static website without database integration.

```bash
nano docker/compose.yaml
```

Add:

```yaml
services:
  app:
    build:
      context: ../app
    container_name: winter-arc-app
    ports:
      - "8080:8080"
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    container_name: winter-arc-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 3s
      retries: 5
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db_data:
```

This example assumes your static-site Dockerfile is in `app/` and serves on container port `8080`.

The website and database share a network, but the static website does not automatically connect to PostgreSQL.

### Step 9: Understand the important Compose settings

| Setting | Purpose |
|---|---|
| `services` | Defines containers in the application stack |
| `app` | Application service |
| `db` | PostgreSQL service |
| `build` | Builds an image using a Dockerfile |
| `image` | Uses an existing image |
| `ports` | Maps host ports to container ports |
| `environment` | Passes configuration variables |
| `depends_on` | Defines service startup dependency |
| `healthcheck` | Checks whether a service is ready |
| `networks` | Allows services to communicate |
| `volumes` | Persists database data |

### Step 10: Understand service-name networking

Inside a Compose network, services can communicate using their service names.

For example:

```text
Application container → db:5432 → PostgreSQL container
```

The application should use:

```text
DB_HOST=db
DB_PORT=5432
```

Do not use:

```text
DB_HOST=localhost
```

Inside a container, `localhost` refers to that same container, not another container.

Docker Compose provides DNS discovery using service names.

### Step 11: Understand named volumes

The Compose file defines:

```yaml
volumes:
  db_data:
```

The database service uses:

```yaml
volumes:
  - db_data:/var/lib/postgresql/data
```

This provides persistent storage for PostgreSQL.

Database data is stored in the named volume rather than only in the container's writable layer.

The volume can survive container removal and recreation.

**Important:** Do not use `docker compose down -v` during the persistence exercise because it removes the Compose-managed named volume.

---

## 4. Task B — Start and Inspect the Stack

### Step 12: Validate Compose configuration

Run from the repository root:

```bash
docker compose --env-file .env -f docker/compose.yaml config
```

This command:

- Reads the Compose YAML file.
- Loads environment variables from `.env`.
- Validates the configuration.
- Displays the expanded configuration.

If an error appears, check YAML indentation, variable names, and file paths.

### Step 13: Build and start services

```bash
docker compose --env-file .env -f docker/compose.yaml up -d --build
```

Explanation:

- `up` creates and starts services.
- `-d` runs services in the background.
- `--build` builds the application image before starting services.

### Step 14: Check service status

```bash
docker compose --env-file .env -f docker/compose.yaml ps
```

You should see both:

- `app`
- `db`

Wait for PostgreSQL's health check to report healthy.

### Step 15: Check logs

View logs for all services:

```bash
docker compose --env-file .env -f docker/compose.yaml logs --tail=100
```

View application logs:

```bash
docker compose --env-file .env -f docker/compose.yaml logs app
```

View database logs:

```bash
docker compose --env-file .env -f docker/compose.yaml logs db
```

Follow logs continuously:

```bash
docker compose --env-file .env -f docker/compose.yaml logs -f
```

Press `Ctrl + C` to stop following logs.

### Step 16: Test the application

Open:

```text
http://localhost:8080
```

Or use:

```bash
curl -I http://localhost:8080
```

A successful web response normally returns:

```text
HTTP/1.1 200 OK
```

### Step 17: Enter the application container

```bash
docker compose --env-file .env -f docker/compose.yaml exec app sh
```

This opens a shell inside the running application container.

Exit:

```bash
exit
```

### Step 18: Inspect Docker networks

```bash
docker network ls
```

Inspect the actual Compose network name shown in the output:

```bash
docker network inspect PROJECT_default
```

Replace `PROJECT_default` with the actual network name.

The application and database should be connected to the same Compose network.

### Step 19: Verify PostgreSQL readiness

```bash
docker compose --env-file .env -f docker/compose.yaml exec db pg_isready -U appuser -d appdb
```

Expected output:

```text
/var/run/postgresql:5432 - accepting connections
```

This confirms that PostgreSQL is ready to accept connections.

For a Node.js application, this check confirms database readiness, but the application must also be configured to connect to `db:5432`.

For a static website, this verifies the database independently; it does not prove website-to-database integration.

---

## 5. Task C — Verify Persistence

The objective is to create a test record in PostgreSQL, restart the Compose stack, and confirm that the record remains.

### Step 20: Enter PostgreSQL

```bash
docker compose --env-file .env -f docker/compose.yaml exec db psql -U appuser -d appdb
```

You are now inside the PostgreSQL command-line interface.

Create a test table:

```sql
CREATE TABLE day33_test (
    id SERIAL PRIMARY KEY,
    message VARCHAR(100)
);
```

Insert a test record:

```sql
INSERT INTO day33_test (message)
VALUES ('Day 33 Docker Compose persistence test');
```

Display the record:

```sql
SELECT * FROM day33_test;
```

You should see the inserted test message.

Exit PostgreSQL:

```sql
\q
```

### Step 21: Identify the named volume

```bash
docker volume ls
```

Find the volume with a name similar to:

```text
projectname_db_data
```

Inspect it:

```bash
docker volume inspect projectname_db_data
```

Replace the example name with the actual volume name shown by your command.

### Step 22: Stop and remove containers without deleting the volume

```bash
docker compose --env-file .env -f docker/compose.yaml down
```

This removes:

- Compose containers.
- Compose network.

It retains the named volume by default.

**Do not use `docker compose down -v` for this test.**

### Step 23: Start the stack again

```bash
docker compose --env-file .env -f docker/compose.yaml up -d
```

Check service status:

```bash
docker compose --env-file .env -f docker/compose.yaml ps
```

Wait until PostgreSQL becomes healthy.

### Step 24: Verify the test record

Enter PostgreSQL again:

```bash
docker compose --env-file .env -f docker/compose.yaml exec db psql -U appuser -d appdb
```

Run:

```sql
SELECT * FROM day33_test;
```

The previous test record should still exist.

Exit:

```sql
\q
```

This demonstrates that the named volume preserves database data after container removal and recreation.

---

## 6. Task D — Safe Break/Fix Troubleshooting Drill

The objective is to create one controlled issue, investigate it, identify the root cause, fix it, and verify recovery.

### Step 25: Confirm services are running

```bash
docker compose --env-file .env -f docker/compose.yaml ps
```

Check the application:

```bash
curl -I http://localhost:8080
```

Check PostgreSQL:

```bash
docker compose --env-file .env -f docker/compose.yaml exec db pg_isready -U appuser -d appdb
```

### Step 26: Stop the database service

```bash
docker compose --env-file .env -f docker/compose.yaml stop db
```

This stops PostgreSQL without deleting its container or volume.

### Step 27: Observe the problem

Check the service status:

```bash
docker compose --env-file .env -f docker/compose.yaml ps
```

Check database logs:

```bash
docker compose --env-file .env -f docker/compose.yaml logs db
```

If your application connects to PostgreSQL, inspect application logs:

```bash
docker compose --env-file .env -f docker/compose.yaml logs app
```

The application may report a database connection error.

If your application is a static website, it may continue working because it does not depend on PostgreSQL.

### Step 28: Identify the root cause

The database service was deliberately stopped.

A database-dependent application cannot access PostgreSQL while the database is unavailable.

### Step 29: Restore the database service

```bash
docker compose --env-file .env -f docker/compose.yaml start db
```

Check service status:

```bash
docker compose --env-file .env -f docker/compose.yaml ps
```

Wait for PostgreSQL to become healthy.

Verify readiness:

```bash
docker compose --env-file .env -f docker/compose.yaml exec db pg_isready -U appuser -d appdb
```

If your application supports database integration, inspect its logs and verify that it can reconnect.

### Step 30: Verify recovery and data persistence

Enter PostgreSQL:

```bash
docker compose --env-file .env -f docker/compose.yaml exec db psql -U appuser -d appdb
```

Run:

```sql
SELECT * FROM day33_test;
```

Exit:

```sql
\q
```

Check the application:

```bash
curl -I http://localhost:8080
```

The database should be ready again, and the previous test record should remain.

### Break/Fix Incident Summary

| Stage | Description |
|---|---|
| Symptom | Database service unavailable |
| Evidence | Compose status showed database stopped |
| Root Cause | PostgreSQL service was deliberately stopped |
| Fix | Restarted database using `docker compose start db` |
| Verification | Database became ready and test data remained |

---

## 7. Troubleshooting Reference

### Problem 1: Application cannot connect to the database

Check:

```bash
docker compose --env-file .env -f docker/compose.yaml ps
docker compose --env-file .env -f docker/compose.yaml logs app
docker compose --env-file .env -f docker/compose.yaml logs db
```

Verify:

- Database hostname is `db`.
- Both services use the same network.
- Database credentials are correct.
- PostgreSQL is healthy.
- Database port is `5432`.

Remember: Use `db`, not `localhost`, for communication between Compose services.

### Problem 2: Database starts slowly

Check:

```bash
docker compose --env-file .env -f docker/compose.yaml ps
docker compose --env-file .env -f docker/compose.yaml logs db
```

Use a health check and `depends_on` with `condition: service_healthy`.

`depends_on` alone may only control startup order. It does not necessarily guarantee that the database is ready.

### Problem 3: Database data disappears

Check:

```bash
docker volume ls
docker volume inspect projectname_db_data
```

Replace the example volume name with your actual volume.

Verify the Compose volume configuration:

```yaml
volumes:
  db_data:
```

And the database mount:

```yaml
volumes:
  - db_data:/var/lib/postgresql/data
```

Avoid `docker compose down -v` unless you intentionally want to delete the named volume and its data.

### Problem 4: Port already in use

Check:

```bash
docker ps
```

If host port `8080` is occupied, change the host port:

```yaml
ports:
  - "8081:3000"
```

The host port becomes `8081`, while the application continues listening on container port `3000`.

### Problem 5: Compose configuration error

Run:

```bash
docker compose --env-file .env -f docker/compose.yaml config
```

Check:

- YAML indentation.
- Service names.
- Environment-variable names.
- File paths.
- Volume configuration.
- `.env` file values.

---

## 8. Useful Docker Compose Commands Reference

| Command | Purpose |
|---|---|
| `docker compose version` | Check Compose version |
| `docker compose config` | Validate Compose configuration |
| `docker compose up -d` | Start services in background |
| `docker compose up -d --build` | Build and start services |
| `docker compose ps` | Check service status |
| `docker compose logs` | View service logs |
| `docker compose logs app` | View application logs |
| `docker compose logs db` | View database logs |
| `docker compose logs -f` | Follow logs |
| `docker compose exec app sh` | Open shell in application container |
| `docker compose exec db sh` | Open shell in database container |
| `docker compose stop db` | Stop database service |
| `docker compose start db` | Start stopped database service |
| `docker compose restart` | Restart services |
| `docker compose down` | Stop and remove containers and network |
| `docker network ls` | List Docker networks |
| `docker volume ls` | List Docker volumes |
| `docker volume inspect name` | Inspect a named volume |

---

## 9. Interview Warm-up — Day 33

### Q1. What problem does Docker Compose solve?

**Answer:** Docker Compose defines and runs a multi-container application as one coordinated stack. It manages services, networks, volumes, and configurations together.

### Q2. How do Compose services find each other?

**Answer:** Services on a shared Compose network communicate using their service names as DNS hostnames.

For example:

```text
DB_HOST=db
DB_PORT=5432
```

### Q3. Why do we use a named volume?

**Answer:** A named volume stores data outside a container's writable layer and allows data to survive container replacement.

### Q4. Does `depends_on` guarantee that the database is ready?

**Answer:** Not by itself. It can control startup order, but readiness requires a health check with `condition: service_healthy` or application-side retry logic.

### Q5. What is a Compose service?

**Answer:** A service is a definition of a containerized component of an application, such as a web application or database.

### Q6. What is the purpose of `compose.yaml`?

**Answer:** It defines application services, images or build configuration, networks, volumes, ports, environment variables, and runtime settings.

### Q7. What is the difference between `docker run` and `docker compose up`?

**Answer:** `docker run` starts an individual container, while `docker compose up` starts the services defined in a Compose configuration.

### Q8. Why should we use `db` instead of `localhost` for the database hostname?

**Answer:** Inside a container, `localhost` refers to that same container. Compose provides service-name DNS, so the application uses `db` to reach the database container.

### Q9. What is the purpose of a health check?

**Answer:** A health check tests whether a service is ready or functioning as expected. For PostgreSQL, `pg_isready` checks whether the database accepts connections.

### Q10. What happens when we run `docker compose down`?

**Answer:** It stops and removes Compose containers and networks. Named volumes are retained by default.

### Q11. What is the difference between `docker compose down` and `docker compose down -v`?

**Answer:** `docker compose down` removes containers and networks while retaining named volumes. Adding `-v` also removes Compose-managed named volumes and can delete persistent database data.

### Q12. How do you check logs for a specific Compose service?

**Answer:**

```bash
docker compose logs app
```

Or:

```bash
docker compose logs db
```

### Q13. What is the purpose of `.env` in Docker Compose?

**Answer:** It supplies configuration values used for variable substitution. It can hold local credentials, but the actual `.env` file should not be committed to Git.

### Q14. Why should the database port generally not be published to the host?

**Answer:** The application can communicate with the database through the internal Compose network. Keeping the database port unpublished reduces unnecessary external exposure.

### Q15. How would you troubleshoot an application that cannot connect to PostgreSQL?

**Answer:** I would check application and database logs, database health, service-name hostname, shared network, database port, and credentials. I would also verify that the application uses `db` instead of `localhost`.

---

## 10. Key Learnings

By completing Day 33, I practiced:

1. Defining multiple services using Docker Compose.
2. Reusing the application Dockerfile from Day 32.
3. Configuring application and database services.
4. Understanding Compose service-name networking.
5. Configuring environment variables and local secrets.
6. Creating a shared internal network.
7. Using PostgreSQL health checks.
8. Understanding named volumes.
9. Demonstrating database persistence after container recreation.
10. Inspecting Compose status and service logs.
11. Performing a controlled database failure and recovery.
12. Understanding the difference between container storage and persistent storage.

**Day 33 Finish Line:**

Multi-Service Stack Running → Network Verified → Database Healthy → Data Persists → One Issue Fixed.

**Learn → Build → Break → Fix → Explain**

---

**Important Note:** If the Day 30 application is a static website, PostgreSQL is a separate supporting service unless actual database integration has been implemented and tested.
