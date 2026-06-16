# Conduit RealWorld App — Dockerized & CI/CD with Azure

A fully containerized fullstack application based on the [RealWorld](https://realworld.io/) spec, featuring an automated CI/CD pipeline built with Azure DevOps, Azure Container Registry, and Azure Container Instances.

> Built as a DevOps learning project to practice Docker, Docker Compose, and Azure Pipelines end-to-end.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React + Vite + SWC |
| Backend | Node.js + Express + Sequelize |
| Database | PostgreSQL |
| Containerization | Docker + Docker Compose |
| CI/CD | Azure Pipelines |
| Container Registry | Azure Container Registry (ACR) |
| Deployment | Azure Container Instances (ACI) |

---

## Architecture┌─────────────────────────────────────────────────────┐

│                  Local Development                  │

│                                                     │

│   ┌──────────┐    ┌──────────┐    ┌──────────────┐ │

│   │ frontend │    │ backend  │    │  PostgreSQL   │ │

│   │  :8080   │───▶│  :3001   │───▶│    :5432     │ │

│   │  nginx   │    │ express  │    │  (named vol) │ │

│   └──────────┘    └──────────┘    └──────────────┘ │

│                Docker Compose                       │

└─────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────┐

│                  CI/CD Pipeline                     │

│                                                     │

│   GitHub Push                                       │

│       │                                             │

│       ▼                                             │

│   Azure Pipelines (Build Stage)                     │

│   ├── docker build frontend image                   │

│   ├── docker build backend image                    │

│   └── docker push → Azure Container Registry        │

│       │                                             │

│       ▼                                             │

│   Azure Pipelines (Deploy Stage)                    │

│   ├── az container create → conduit-backend (ACI)   │

│   └── az container create → conduit-frontend (ACI)  │

│                                                     │

└─────────────────────────────────────────────────────┘

---

## CI/CD Pipeline

The pipeline is defined in `azure-pipelines.yml` and triggers automatically on every push to `main`.

### Build Stage
- Logs into Azure Container Registry using admin credentials stored as secret pipeline variables
- Builds the backend Docker image from `backend/Dockerfile`
- Builds the frontend Docker image from `frontend/Dockerfile` using a multi-stage build (Node builder + nginx serve)
- Tags each image with both the build ID and `latest`
- Pushes both images to ACR

### Deploy Stage
- Authenticates with Azure using a service principal via the `conduit-azure` service connection
- Deploys the backend container to Azure Container Instances with environment variables injected securely
- Deploys the frontend container to Azure Container Instances exposed on port 80

### Pipeline Variables
Sensitive values are stored in an Azure DevOps variable group (`conduit-secrets`) and never hardcoded in the YAML:
- `ACR_USERNAME` — ACR admin username
- `ACR_PASSWORD` — ACR admin password (secret)
- `JWT_KEY` — application JWT secret (secret)

---

## Running Locally with Docker Compose

### Prerequisites
- Docker Desktop installed and running
- WSL2 (if on Windows)

### Setup

Clone the repository:
```bash
git clone https://github.com/Duubemmm/conduit-realworld-example-app.git
cd conduit-realworld-example-app
```

Create a `.env` file in the project root:
```bash
# App
PORT=3001
JWT_KEY=supersecretkey_changeme

# Database
DEV_DB_USERNAME=conduit
DEV_DB_PASSWORD=conduitpass
DEV_DB_NAME=conduit_dev
DEV_DB_HOSTNAME=db
DEV_DB_DIALECT=postgres
DEV_DB_LOGGING=false
```

Build and start all services:
```bash
docker compose up --build
```

### Services
| Service | URL |
|---|---|
| Frontend | http://localhost:8080 |
| Backend API | http://localhost:3001/api |
| PostgreSQL | localhost:5432 |

### Useful Commands
```bash
# Run in background
docker compose up -d

# View logs
docker compose logs -f

# Stop all containers
docker compose down

# Stop and delete database volume
docker compose down -v
```

---

## Project Structure
conduit-realworld-example-app/

├── backend/

│   ├── Dockerfile

│   ├── .dockerignore

│   ├── config/

│   ├── controllers/

│   ├── middleware/

│   ├── migrations/

│   ├── models/

│   └── routes/

├── frontend/

│   ├── Dockerfile        # Multi-stage build

│   ├── .dockerignore

│   └── src/

├── docker-compose.yml

├── azure-pipelines.yml

└── .env                  # Not committed — see setup above

---

## Key DevOps Concepts Practiced

- **Multi-stage Docker builds** — separate builder and serve stages for the frontend, keeping the final image small
- **Docker Compose orchestration** — health checks, named volumes, service dependencies, and environment variable injection
- **CI/CD pipeline design** — separate Build and Deploy stages with a clear dependency chain
- **Secret management** — sensitive values stored in Azure DevOps variable groups, never in source code
- **Container registry workflow** — building, tagging with build ID, and pushing images to a private registry
- **Cloud deployment** — deploying containers to Azure Container Instances via Azure CLI in the pipeline

---

## Azure Pipeline

[View Pipeline runs on Azure DevOps](https://dev.azure.com/Chidubem-Okoli/conduit/_build)

---

## Original Application

This project is based on the [RealWorld example app](https://github.com/TonyMckes/conduit-realworld-example-app) by TonyMckes, used as a base to practice DevOps tooling. All Docker, CI/CD, and Azure infrastructure work was added independently.