# Practicum 3 — Exercise Answers

## E3 — Rolling Database Migration

The auditing migration is divided into two steps.

### V4 — Expand and Backfill

V4 adds the created_by and last_modified_by columns as nullable fields. It then updates existing records with the value 'unknown'.

Keeping the columns nullable allows older application instances to continue inserting books during a rolling deployment, even though those instances do not provide the new auditing fields.

### V5 — Enforce Constraints

V5 changes created_by and last_modified_by to NOT NULL.

This migration must be deployed in a later release, after all running application instances have been updated to write both fields.

### Why Not Deploy Both Together?

During a rolling upgrade, old and new application versions may run at the same time.

If the NOT NULL constraints are added immediately, older instances may fail when inserting records because they do not supply the auditing fields.

Separating the migrations prevents this compatibility problem and supports a safer rolling deployment.

### Verification

- Flyway migrations V1 through V5 applied successfully in local development.
- Existing records were backfilled before applying NOT NULL constraints.
- Keycloak authentication and token relay were tested through Edge Service.
- Creating a book as isabelle returned HTTP 201.
- PostgreSQL confirmed created_by = isabelle and last_modified_by = isabelle.

## Bonus Questions — Practicum 3

### 1. Why does deleting a secret file in a later Dockerfile instruction not remove it from the image?

Docker images are built in layers, so deleting a secret later does not completely remove it. The secret can still be found in an earlier layer. It is better to avoid putting secrets in the image at all.

### 2. What is one advantage of Kubernetes Services and Eureka?

Kubernetes Services give Pods a stable address, so other applications do not need to keep track of changing Pod IP addresses.

Eureka lets applications find available service instances and choose which one to use. This gives the client more control over where requests are sent.

### 3. What is the difference between an ID token and an access token?

An ID token tells the client application who the user is. For example, Edge Service can use it to identify the logged-in user.

An access token is used to access protected resources. For example, Edge Service sends it to Catalog Service to prove that the request is authorized.

### 4. Why should Edge Service store authorized clients in the web session instead of memory?

If Edge Service has two replicas, a user's requests might go to either Pod. If the login information and tokens are only stored in one Pod's memory, the other Pod will not have them.

This can cause token relay to fail when a request goes to the other Pod. Using Redis sessions allows both Pods to access the same session information.

### 5. How long can a disabled employee's access token still be used?

The access token can still work until it expires, which is up to 5 minutes in this project.

This happens because Catalog Service checks the JWT's signature and expiration instead of asking Keycloak whether the account is still enabled for every request. Disabling the account does not immediately invalidate an existing token.

### 6. Why does Catalog Service return 401 sometimes and 403 other times?

A 401 means the request is not properly authenticated, such as when the access token is missing or invalid.

A 403 means the user is authenticated but does not have permission to perform the action. For example, a customer cannot add books if that action requires the employee role.

Spring Security in Catalog Service handles these authentication and authorization checks.

