---
layout: page
title: "Docker: Containers from First Principles"
week_number: "41"
release_date: "2027-01-26"
phase_label: "Phase 9"
phase_name: "DevOps & Cloud Security"
phase_color: "#4cc9f0"
permalink: /week-41/
---

> Week 41. Phase 9: DevOps and infrastructure security.
> 
> Docker is how modern applications are packaged and deployed. Every cloud service, every lab VM alternative, every CI/CD pipeline uses containers. Security professionals need to understand Docker because containers are everywhere -- and they can be misconfigured.
> 
> Install Docker Desktop from https://www.docker.com/products/docker-desktop before this session.

---

# Docker Intro: Containers from First Principles

**Domain:** devops
**Difficulty:** beginner
**Estimated time:** 60 minutes

## Objectives

- Run, inspect, and manage Docker containers using the CLI
- Write a multi-stage `Dockerfile` that builds a minimal Python app image
- Use Docker Compose to run two services (app + Redis) with a shared network
- Understand the difference between images, containers, layers, and volumes

## Prerequisites

- Docker Desktop installed (Windows/macOS) or Docker Engine (Linux)
- Basic command-line familiarity
- Python knowledge helpful but not required

## Materials / Tools

- `starter/app.py` — Flask hit-counter backed by Redis
- `starter/Dockerfile` — multi-stage build skeleton
- `starter/docker-compose.yml` — two-service compose file skeleton

```bash
docker --version          # verify ≥ 24.0
docker compose version    # verify ≥ 2.0
```

## ETHICS & SAFETY

> Docker pulls images from Docker Hub. Only pull images from official or verified publishers.

- Never run containers as root in production (`USER` instruction in Dockerfile).
- Be cautious with `--privileged` containers — they can escape container isolation.
- Do not expose container ports to `0.0.0.0` on shared networks without firewall rules.

## Instructions

### Part 1 — Run Your First Container (10 min)

```bash
# Pull and run a hello-world container
docker run hello-world

# Run an interactive Ubuntu shell
docker run -it --rm ubuntu:22.04 bash
# Inside: apt-get update && apt-get install -y curl && curl http://example.com

# List running and stopped containers
docker ps
docker ps -a

# Clean up stopped containers
docker container prune
```

### Part 2 — Write the Dockerfile (20 min)

Open `starter/Dockerfile` and complete the two-stage build:

```dockerfile
# Stage 1: builder — install dependencies
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: runtime — copy only what is needed
FROM python:3.12-slim
WORKDIR /app
# TODO: Copy installed packages from builder stage
# COPY --from=builder /root/.local /root/.local
# TODO: Copy application code
# COPY app.py .
# TODO: Run as non-root user
# RUN useradd -m appuser && chown -R appuser /app
# USER appuser
ENV PORT=8000
EXPOSE 8000
CMD ["python", "app.py"]
```

Build and test:

```bash
docker build -t myapp:dev starter/
docker run --rm -p 8000:8000 myapp:dev
curl http://localhost:8000/
```

### Part 3 — Docker Compose (15 min)

Open `starter/docker-compose.yml` and complete it:

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - REDIS_HOST=redis
    depends_on:
      - redis

  redis:
    image: redis:7-alpine
    # TODO: Add a named volume for redis data persistence
    volumes:
      - redis_data:/data

volumes:
  redis_data:
```

Run:

```bash
docker compose -f starter/docker-compose.yml up --build
curl http://localhost:8000/          # visit counter increments
docker compose -f starter/docker-compose.yml down -v
```

### Part 4 — Inspect and Debug (10 min)

```bash
# Image layers
docker history myapp:dev

# Container filesystem
docker run --rm myapp:dev ls -la /app

# Live logs
docker compose logs -f app

# Exec into a running container
docker compose exec app bash

# Resource usage
docker stats --no-stream
```

### Part 5 — Clean Up

```bash
docker compose down -v
docker image rm myapp:dev
docker system prune -f
```

## Expected Output

```
$ curl http://localhost:8000/
{"visits": 1, "message": "Hello from Docker!"}

$ curl http://localhost:8000/
{"visits": 2, "message": "Hello from Docker!"}
```

## Quick Quiz

1. What is the difference between `CMD` and `ENTRYPOINT` in a Dockerfile?
2. Why does `.dockerignore` matter for build speed and image security?
3. Two containers on the same Docker Compose network — can they communicate by service name? How?

*(Answers in `solution/quiz_answers.md`)*

## Extensions

- Add a `healthcheck` to the Dockerfile: `HEALTHCHECK CMD curl -f http://localhost:8000/health || exit 1`.
- Add a `/health` route to `app.py` that pings Redis and returns `{"status": "ok"}` or `{"status": "degraded"}`.
- Use `dive` (free tool) to inspect image layer sizes and identify what is bloating your image.

## Rubric

| Criterion | Pass |
|-----------|------|
| Dockerfile | Multi-stage build completes without errors, image < 200MB |
| Compose | `docker compose up` starts both services, visit counter increments |
| Non-root | Container process runs as non-root user (verify with `docker compose exec app whoami`) |
| Cleanup | `docker compose down -v` removes containers and volumes cleanly |

## Files Produced

```
starter/Dockerfile         (completed)
starter/docker-compose.yml (completed)
```

---

## Activity: Docker Intro Activity

1. Work through the activity at activities/devops/beginner/02_docker-intro.md
1. Run your first container: docker run hello-world
1. Pull and run nginx: docker run -d -p 8080:80 nginx -- verify it serves a page at http://localhost:8080
1. Write a Dockerfile for your Python recon tool from Week 30. It should run the tool when the container starts
1. Build it: docker build -t recon-tool . -- run it: docker run recon-tool example.com
1. Push your image to Docker Hub (create a free account): docker tag recon-tool yourusername/recon-tool && docker push yourusername/recon-tool
1. You know you are done when your recon tool runs inside a container and your image is on Docker Hub

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between a Docker image and a Docker container?
2. What does the -p 8080:80 flag do in docker run?
3. What is the purpose of a Dockerfile?
4. Why are containers more portable than installing software directly on a machine?
