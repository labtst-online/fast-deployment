# FastBoosty Deployment

This repository contains the Docker Compose configuration to deploy and run the entire FastBoosty application stack, including all microservices, databases, caches, and the frontend.

## Overview

The FastBoosty application is built using a microservices architecture. This deployment setup uses Docker and Docker Compose to orchestrate the following services:

*   **Backend Services (FastAPI):**
    *   [`auth_service`](https://github.com/fotapol/fastboosty-auth_service): Handles user authentication and authorization.
    *   [`profile_service`](https://github.com/fotapol/fastboosty-profile_service): Manages user profiles.
    *   [`content_service`](https://github.com/fotapol/fastboosty-content_service): Manages user-generated content (posts).
    *   [`subscription_service`](https://github.com/fotapol/fastboosty-subscription_service): Manages subscription tiers and user subscriptions.
    *   [`payment_service`](https://github.com/fotapol/fastboosty-payment_service): Handles payment processing (integrates with Stripe).
*   **Frontend (React):**
    *   [`frontend prototype`](https://github.com/fotapol/fastboosty-frontend): A React application providing the user interface (served via Nginx).
*   **Infrastructure:**
    *   `nginx`: Acts as a reverse proxy and API gateway.
    *   `db` (PostgreSQL): Primary relational database for the services.
    *   `redis`: In-memory cache (used by `profile_service`).
    *   `zookeeper` & `kafka`: Message queuing system (used by `payment_service` and `subscription_service`).

## Prerequisites

* Git
* Docker
* Docker Compose

## Getting Started

1.  **Clone Repositories:**
    *   Clone this deployment repository:
        ```bash
        git clone https://github.com/fotapol/fastboosty-deployment.git
        cd fastboosty-deployment
        ```
    *   If you intend to build services locally (using the override file), clone all the individual service repositories (`fastboosty-auth_service`, `fastboosty-profile_service`, etc.) into the same parent directory as `fastboosty-deployment`. The `docker-compose.override.yaml` expects this structure:
        ```
        parent-directory/
        ├── fastboosty-deployment/
        ├── fastboosty-auth_service/
        ├── fastboosty-profile_service/
        ├── fastboosty-content_service/
        ├── fastboosty-subscription_service/
        ├── fastboosty-payment_service/
        └── fastboosty-frontend/
        ```

2.  **Configuration:**
    *   This deployment expects environment variables to be defined in files within a `.envs` directory inside `fastboosty-deployment`.
    *   Create the `.envs` directory: `mkdir .envs`
    *   For each service (`auth`, `profile`, `content`, `subscription`, `payment`), copy its corresponding `.env.sample` file from its respective service repository into the `.envs` directory and rename it (e.g., `cp ../fastboosty-auth_service/.env.sample .envs/auth.env`).
    *   Review and **modify the variables** within each `.env` file in the `.envs` directory according to your environment (database credentials, API keys like Stripe, JWT secrets, etc.).
    *   Ensure the `POSTGRES_USER` and `POSTGRES_PASSWORD` variables in your service `.env` files match the ones used by the `db` service (either default `postgres`/`postgres` or set via environment variables when running `docker-compose`).

## Running the Application

You have two main options to run the application:

**Option 1: Using Pre-built Images**

This uses the pre-build images specified in `docker-compose.yaml` (pulled from ghcr.io).

```bash
docker-compose -f docker-compose.yaml up -d
```

**Option 2: Building Services Locally (for development)**

This uses `docker-compose.override.yaml` to build the service images from their local source code. Ensure you have cloned all service repositories as described in "Getting Started".

```bash
docker-compose -f docker-compose.yaml -f docker-compose.override.yaml up --build -d
```

## Accessing Services

Once the containers are up and running:

*   **Frontend:** Access the application via your browser at `http://localhost:3000` (served by Nginx).
*   **API Gateway:** APIs are routed through Nginx at `http://localhost:8080/api/`. Examples:
    *   Auth Service: `http://localhost:8080/api/auth/...`
    *   Profile Service: `http://localhost:8080/api/profile/...`
    *   Content Service: `http://localhost:8080/api/content/...`
    *   Subscription Service: `http://localhost:8080/api/subscription/...`
    *   Payment Service: `http://localhost:8080/api/payment/...`

## Stopping the Application

To stop and remove the containers, networks, and volumes created by docker-compose:

```bash
# If you used Option 1:
docker-compose -f docker-compose.yaml down

# If you used Option 2:
docker-compose -f docker-compose.yaml -f docker-compose.override.yaml down
```

## License

This repository is licensed under the terms of the MIT license.
