# Polar Bookshop — Containers

## Overview

Polar Bookshop uses Docker containers to run the application and its supporting services.

The main components are:

- Catalog Service: Spring Boot application for managing books.
- Edge Service: API gateway that handles routing, authentication, and resilience.
- PostgreSQL: Stores the book catalog.
- Redis: Stores sessions and supports API rate limiting.
- Keycloak: Provides OAuth2 and OpenID Connect authentication.

## Container Images

The application images are built using Spring Boot Buildpacks and published to GitHub Container Registry.

Catalog Service:

ghcr.io/p466-heer/catalog-service:latest

Edge Service:

ghcr.io/p466-heer/edge-service:latest

Catalog Service also includes a Dockerfile.

## Running with Docker Compose

From the polar-deployment repository, run:

docker compose -f docker/docker-compose.yml up -d

Check the running containers:

docker ps

Stop the containers:

docker compose -f docker/docker-compose.yml down

## Building Images Locally

From the catalog-service repository:

./gradlew bootBuildImage --imageName=ghcr.io/p466-heer/catalog-service:latest

From the edge-service repository:

./gradlew bootBuildImage --imageName=ghcr.io/p466-heer/edge-service:latest

## Continuous Integration

Both application repositories use GitHub Actions to:

1. Build and test the application.
2. Validate Kubernetes manifests.
3. Scan for vulnerabilities.
4. Build a container image.
5. Publish the image to GitHub Container Registry.

## Verification

Check container status:

docker ps

Check Catalog Service health when it is accessible on localhost:9001:

curl http://localhost:9001/actuator/health
