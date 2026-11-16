# Polar Bookshop — Kubernetes Deployment

## Overview

Polar Bookshop runs on a local Kubernetes cluster using Minikube.

The application consists of:
- Catalog Service: Spring Boot REST API
- Edge Service: Spring Cloud Gateway
- PostgreSQL: Catalog database
- Redis: Session storage and rate limiting
- Keycloak: OAuth2/OpenID Connect identity provider

## 1. Start Minikube

Start the local Kubernetes cluster:

    minikube start -p polar --driver=docker

Select the cluster:

    kubectl config use-context polar

Check cluster status:

    kubectl get nodes

## 2. Deploy Platform Services

From the polar-deployment repository, deploy PostgreSQL:

    kubectl apply -f kubernetes/postgres.yml

Deploy Redis:

    kubectl apply -f kubernetes/platform/development/services/redis.yml

Check the pods:

    kubectl get pods

## 3. Deploy Catalog Service

Apply the Catalog Service manifest:

    kubectl apply -f ../catalog-service/k8s/catalog-service.yml

Verify the deployment:

    kubectl get deployment catalog-service

    kubectl get service catalog-service

## 4. Deploy Edge Service

Apply the Edge Service manifests:

    kubectl apply -f ../edge-service/k8s/deployment.yml

    kubectl apply -f ../edge-service/k8s/service.yml

Verify the deployment:

    kubectl get deployment edge-service

    kubectl get service edge-service

## 5. Keycloak Authentication

Keycloak runs locally in Docker on port 8080.

Realm: PolarBookshop

Client: edge-service

The Edge Service uses OpenID Connect for login and JWT token relay to Catalog Service.

For this local Minikube setup, the Edge Service deployment includes a proxy sidecar that forwards Keycloak traffic to the Docker host.

## 6. Access the Application

Forward the Edge Service port:

    kubectl port-forward service/edge-service 9004:80

Open the login endpoint:

    http://localhost:9004/oauth2/authorization/keycloak

After login, access:

    http://localhost:9004/user

The user endpoint returns the authenticated username and roles.

## 7. Health Checks

Check running pods:

    kubectl get pods

Check deployments:

    kubectl get deployments

Check services:

    kubectl get services

Check the Edge Service health endpoint:

    curl http://localhost:9004/actuator/health

Catalog Service also provides Kubernetes liveness and readiness probes.

## 8. Graceful Shutdown

Both application deployments use a Kubernetes preStop lifecycle hook.

Spring Boot is configured for graceful shutdown, allowing active requests to finish before termination.

## 9. Tilt

Tiltfiles are included in the application repositories and polar-deployment.

To start Tilt from polar-deployment:

    tilt up

Tilt applies the configured Kubernetes resources and provides a local development dashboard.

## 10. Troubleshooting

View pods:

    kubectl get pods

Describe a pod:

    kubectl describe pod <pod-name>

View logs:

    kubectl logs deployment/catalog-service

    kubectl logs deployment/edge-service -c edge-service

Restart deployments:

    kubectl rollout restart deployment/catalog-service

    kubectl rollout restart deployment/edge-service

Stop Minikube:

    minikube stop -p polar

## Exercise E1 — Readiness vs. Liveness

We configured two Catalog Service replicas and tested how Kubernetes responds when PostgreSQL becomes unavailable.

### Experiment 1: Database in Readiness

Configuration:
- readiness: readinessState,db
- liveness: livenessState

Before the outage:
- Catalog Service: 2/2 ready
- Service endpoints: 2

After scaling PostgreSQL to zero:
- Catalog Service: 0/2 ready
- Service endpoints: 0
- Both Pods remained Running
- Container restarts: 0

After restoring PostgreSQL:
- Catalog Service returned to 2/2 ready
- Both Service endpoints returned automatically

### Experiment 2: Database in Liveness

Configuration:
- readiness: readinessState
- liveness: livenessState,db

After scaling PostgreSQL to zero:
- Both Catalog Pods became unready
- Service endpoints dropped to zero
- Both containers repeatedly restarted
- Each container reached 3 restarts during observation

### Explanation

Readiness determines whether a Pod should receive traffic. When PostgreSQL becomes unavailable, Catalog Service cannot handle database-dependent requests, so Kubernetes should remove it from the Service endpoints without restarting it.

Liveness determines whether Kubernetes should restart a container. Including PostgreSQL in liveness is worse because restarting Catalog Service cannot fix an unavailable database. It causes unnecessary restarts and may delay recovery.

Therefore, database health belongs in readiness, not liveness.
