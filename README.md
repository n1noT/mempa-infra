# 🐳 Mempa - Infrastructure

This repository contains the Docker infrastructure configuration (`docker-compose.yml`) to orchestrate the complete development environment for the **Mempa** project.

⚠️ **Important:** This repository does not work alone. It is designed to work in synergy with the application repositories:
- [mempa-front](https://github.com/HanTy63/mempa-front) (Angular Application)
- [mempa-api](https://github.com/n1noT/mempa-api) (Express.js / Prisma API)

---

## 📂 Required Folder Architecture

For Docker volumes (Bind Mounts) to work correctly, your three repositories must strictly be cloned into the same parent folder, according to the following structure:

```text
📁 your-workspace-folder/
 ├── 📁 mempa-api/      # Backend API source code
 ├── 📁 mempa-front/    # Angular Frontend source code
 └── 📁 mempa-infra/    # This repository (Docker files)
```

---

## 🚀 Services included in the infrastructure

This repository is split into two architectures depending on your needs.

### 1. Main Application (`docker-compose.yml`)
Runs the main app stack.

| Service | Technology | Default Port | Description |
| :--- | :--- | :--- | :--- |
| **mempa-front** | Node/Angular | `4200` | User interface (Hot-reload enabled). |
| **mempa-api** | Node/Express | `3000` | Backend API connected to Prisma. |
| **mempa-db** | PostgreSQL 16 | `5432` | Main application database. |

### 2. Code Quality & Analysis (`docker-compose.sonar.yml`)
Isolated tools specifically for code analysis. You only need to run this when performing analysis to save system resources.

| Service | Technology | Default Port | Description |
| :--- | :--- | :--- | :--- |
| **mempa-sonarqube**| SonarQube | `9000` | Continuous code quality analysis tool. |
| **mempa-sonarqube-db**| PostgreSQL 16 | *(Internal)* | Dedicated isolated database for SonarQube. |

---

## 🛠️ Installation and Launch Guide (Linux/WSL Environment)

The current architecture uses **full Bind Mounts** for real-time synchronization with your host machine (ideal on Linux/WSL). 

### 1. Environment Preparation
Make sure you have a `.env` file at the root of `mempa-infra/`. It must contain the necessary variables for `docker-compose.yml`:
```env
# Example .env for infra
FRONT_PORT=4200
FRONT_NODE_ENV=development

BACK_PORT=3000
BACK_NODE_ENV=development

# Database Variables
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=secret
DB_NAME=mempa_db

# URL used for Prisma
DATABASE_URL="postgresql://${DB_USER}:${DB_PASSWORD}@mempa-db:5432/${DB_NAME}?schema=public"
```

You must also initialize the API's `.env` file by copying the `.env.example` file provided in the `mempa-api` repository and update it with the correct `DATABASE_URL` (which should point to the `mempa-db` service):
```bash
cd ../mempa-api
cp .env.example .env
```

### 2. Installing Dependencies (On the host machine)
Since the code and dependencies are synchronized, you will need to install the packages on your computer before launching Docker:
```bash
# For the backend
cd ../mempa-api
npm install

# For the frontend
cd ../mempa-front
npm install
```

### 3. Launching the infrastructure

**To run the main application:**
Go back to the `mempa-infra` folder and start the containers:
```bash
cd ../mempa-infra
docker compose up -d --build
```
*(The `--build` flag ensures that Docker images are recreated according to the latest `Dockerfile` modifications).*

**To run Code Quality Analysis (SonarQube):**
When you are ready to perform a code scan, you can launch the isolated SonarQube environment:
```bash
docker compose -f docker-compose.sonar.yml up -d
```
*(To save system resources, you can stop it when your analysis is done by running `docker compose -f docker-compose.sonar.yml down`)*.

Then you can access SonarQube at `http://localhost:9000` and use the default credentials (`admin`/`admin`) to log in.

### 4. Database Initialization (Prisma)
Once the PostgreSQL database is started, apply the Prisma migrations and generate the client locally (from your machine):
```bash
cd ../mempa-api
npx prisma migrate dev
```

---

## 🛑 Stopping the infrastructure
To gracefully stop all services without deleting the database:
```bash
docker compose down
```

*(If you want to reset everything, including database volumes, add the `-v` flag).*

