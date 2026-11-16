# Polar Bookshop — Practicum 3 Demo

## Overview

This document describes how to demonstrate the Docker, Kubernetes, API Gateway, and Keycloak features of Polar Bookshop.

## 1. Verify Kubernetes

Check the running pods:

    kubectl get pods

Check deployments and services:

    kubectl get deployments
    kubectl get services

Expected components:
- catalog-service
- edge-service
- polar-postgres
- polar-redis

## 2. Access Edge Service

Forward the Edge Service port:

    kubectl port-forward service/edge-service 9004:80

Open the application login page:

    http://localhost:9004/oauth2/authorization/keycloak

Log in with a Keycloak test user.

After authentication, visit:

    http://localhost:9004/user

Verify that the response contains the authenticated username and roles.

## 3. Verify Catalog Routing

Access the book catalog through Edge Service:

    http://localhost:9004/books

Confirm that the gateway routes the request to Catalog Service and returns book data.

## 4. Verify Role-Based Authorization

The application uses Keycloak roles:

- customer
- employee

Customers can view books.

Employees can create, update, and delete books.

Attempting a protected operation without the employee role should return HTTP 403.

## 5. Verify User Auditing

Catalog Service records:

- created_date
- last_modified_date
- created_by
- last_modified_by
- version

Create or update a book using an authenticated employee account.

Inspect the database to confirm that the auditing fields contain the expected values.

## 6. Verify Health Checks

Check Edge Service health:

    curl http://localhost:9004/actuator/health

Check Kubernetes pod readiness:

    kubectl get pods

Catalog Service and Edge Service use Kubernetes liveness and readiness probes.

## 7. Verify Resilience

Edge Service uses:

- Circuit breaker
- Retry for GET requests
- Redis-backed rate limiting
- Redis-backed HTTP sessions

These features help the application handle temporary service failures and manage incoming traffic.

## 8. Verify CI/CD

Both application repositories include GitHub Actions workflows.

The pipelines are configured to:

1. Build and test the application.
2. Validate Kubernetes manifests.
3. Scan for vulnerabilities.
4. Build container images.
5. Publish images to GitHub Container Registry.

Confirm that the latest workflow runs are successful in GitHub Actions.

## 9. Repository Deliverables

Submit the GitHub repository URLs for:

- catalog-service
- edge-service
- polar-deployment

Verify that the required Kubernetes manifests, Tiltfiles, documentation, tests, and workflows are committed.
