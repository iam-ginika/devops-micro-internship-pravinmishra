# Assignment 7 — Capstone: Deploy a Production-Grade Stack for The EpicBook

Part of the DevOps Micro Internship (DMI) with Agentic AI

---

## Purpose

In this assignment, you will deploy the EpicBook application as a production-oriented Docker Compose stack on a cloud VM. You will use optimized container images, isolated networks, health checks, persistent MySQL storage, a selected reverse proxy, logging, backup and restore testing, and reliability procedures.

---

# Task 0 — App Discovery and Architecture

## Goal

Review the EpicBook repository and design the intended application architecture.

### Evidence

#### Screenshot 1 — EpicBook Project Structure

Add a terminal screenshot showing the EpicBook project structure after cloning the repository.

![Output](screenshots/Project-structure.png)

---

#### Screenshot 2 — Architecture Diagram

Add a screenshot of your architecture diagram showing:

- Public user
- Reverse proxy
- Frontend
- Backend
- Database
- Docker networks
- Public and private ports
- Persistent database storage

Add your full name inside the diagram or as a clear caption below it.

![Output](screenshots/Diagram.webp)

---

#### Screenshot 3 — Environment Variables and Ports Document

Add a screenshot showing the contents of:

```text
docs/02-env-and-ports.md
```

It must document environment-variable names, internal ports, persistent-data details, and the health-check method. Do not expose real credentials or values.

![Output](screenshots/Docs.png)

---

# Task 1 — Create Production Docker Images

## Goal

Create optimized production images for the EpicBook backend and frontend.

### Evidence

#### Screenshot 4 — Backend Dockerfile

Add a screenshot showing `backend/Dockerfile`, including:

- Dependency stage
- Minimal runtime stage
- Production startup command
- Internal backend port
- Non-root user configuration

![Output](screenshots/Backend-Dockerfile.png)

---

#### Screenshot 5 — Frontend Dockerfile

Add a screenshot showing `frontend/Dockerfile`, including:

- Nginx runtime image
- Static frontend files copied to the Nginx web root

![Output](screenshots/Frontend-Dockerfile.png)

---

#### Screenshot 6 — Docker Ignore Files

Add a screenshot showing both:

```text
backend/.dockerignore
frontend/.dockerignore
```

![Output](screenshots/Ignore.png)

---

#### Screenshot 7 — Docker Image Builds and Size Comparison

Add a terminal screenshot showing successful builds of:

- Baseline backend image
- Optimized backend image
- Frontend image

The screenshot must also show the baseline and optimized backend image-size comparison.

![Output](screenshots/Build-success.png)
![Output](screenshots/Docker-Images2.png)

---

#### Screenshot 8 — Backend Running as Non-Root User

Add a terminal screenshot showing the optimized backend container running as a non-root user.

![Output](screenshots/Non-root-user.png)

---

### Notes

Write a short note covering:

- Baseline and optimized backend image sizes
- The image-size reduction achieved
- One Docker layer-caching optimization used
- The security benefit of running the backend as a non-root user

Write your note here.
Baseline and optimized backend image sizes: The baseline backend image (epicbook-backend:baseline) has a content size of 428 MB and uses 1.73 GB of disk. The optimized image (epicbook-backend:optimized) has a content size of 66.3 MB and uses 276 MB of disk.

The image-size reduction achieved: The optimized image is 361.7 MB smaller by content size, an 84.5% reduction. This comes from using an Alpine base image, installing production dependencies only, and removing npm from the final stage.

One Docker layer-caching optimization used: package.json and package-lock.json are copied, and npm ci runs, before the rest of the application source is copied. When only the code changes, Docker reuses the cached dependency layer and doesn’t reinstall the packages.

The security benefit of running the backend as a non-root user: The container runs as the node user (uid 1000), and the application code is owned by root. If the application is compromised, the attacker doesn’t have root privileges inside the container and can’t rewrite the application files, which limits the damage.

---

# Task 2 — Create the Docker Compose Stack and Networks

## Goal

Create one Docker Compose stack containing the reverse proxy, frontend, backend, and MySQL database.

### Evidence

#### Screenshot 9 — Docker Compose Services

Add a screenshot showing `docker-compose.yml` with all four services:

```text
reverse-proxy
frontend
backend
database
```

![Output](screenshots/Services.png)
![Output](screenshots/Services2.png)


---

#### Screenshot 10 — Networks and Named Volume

Add a screenshot showing:

- `front-tier` network
- `back-tier` network
- `db_data` named volume

![Output](screenshots/NV.png)

---

#### Screenshot 11 — Docker Compose Validation

Add a terminal screenshot showing successful Docker Compose validation without exposing environment-variable values or secrets.

![Output](screenshots/Compose-config.png)

---

# Task 3 — Configure Health Checks and Startup Dependencies

## Goal

Configure health checks and ensure services start only after their dependencies are healthy.

### Evidence

#### Screenshot 12 — Backend Health Endpoint

Add a screenshot showing the backend application configuration for the `/health` endpoint.

![Output](screenshots/Health-endpoint.png)

---

#### Screenshot 13 — MySQL and Backend Health Checks

Add a screenshot showing `docker-compose.yml` with health checks for MySQL and the backend.

![Output](screenshots/Database-healthcheck.png)
![Output](screenshots/Backend-healthcheck.png)

---

#### Screenshot 14 — Frontend and Reverse-Proxy Health Checks

Add a screenshot showing:

- Frontend health check
- Reverse-proxy health check
- `depends_on` conditions using `service_healthy`

![Output](screenshots/Healthcheck.png)
![Output](screenshots/Depends-on.png)

---

#### Screenshot 15 — Running Healthy Services

Add a terminal screenshot showing Docker Compose service status. The database, backend, frontend, and reverse proxy must be running successfully.

![Output](screenshots/Compose-status.png)

---

#### Screenshot 16 — Public Health Endpoint

Add a terminal screenshot showing a successful response from the public application health endpoint through the reverse proxy.

![Output](screenshots/Curl.png)

---

#### Screenshot 17 — Health-Check and Startup-Order Document

Add a screenshot showing the contents of:

```text
docs/03-healthchecks-and-depends-on.md
```

Explain the health-check method for each service and the startup dependency order.

![Output](screenshots/Docs-03.png)

---

# Task 4 — Configure the Reverse Proxy and Same-Origin Routing

## Goal

Use either Nginx or Traefik as the only public entry point for the EpicBook application.

### Evidence

#### Screenshot 18 — Selected Reverse-Proxy Configuration

Add a screenshot showing the configuration for your selected reverse proxy.

It must show routes for:

- Static frontend assets
- Application pages
- API requests
- Health endpoint

![Output](screenshots/Proxy1.png)
![Output](screenshots/Proxy2.png)

---

#### Screenshot 19 — Only Reverse Proxy Publishes Port 80

Add a screenshot of `docker-compose.yml` showing that only the `reverse-proxy` service publishes port 80.

![Output](screenshots/Port.png)
![Output](screenshots/Port1.png)

---

#### Screenshot 20 — Reverse-Proxy Route Testing

Add a terminal screenshot showing successful requests through the selected reverse proxy to:

- Application page
- One API endpoint
- One static asset
- Health endpoint

![Output](screenshots/Testing.png)

---

#### Screenshot 21 — EpicBook Application Through Public IP

Add a browser screenshot showing the EpicBook application loaded through the VM public IP address.

Add your full name as a clear caption below the screenshot.

![Output](screenshots/Epicbook-browser.png)

---

#### Screenshot 22 — Proxy Routing and CORS Document

Add a screenshot showing the contents of:

```text
docs/04-proxy-routing-and-cors.md
```

Explain the proxy routes and state whether CORS was required and why.

![Output](screenshots/Docs-04.png)

---

# Task 5 — Prove Data Persistence, Backup, and Restore

## Goal

Verify MySQL persistence and perform a controlled backup and restore drill.

### Evidence

#### Screenshot 23 — MySQL Volume Configuration

Add a terminal screenshot showing the `db_data` named volume and its MySQL mount configuration.

![Output](screenshots/db-data.png)
![Output](screenshots/db-data1.png)

---

#### Screenshot 24 — Test Data Before Backup

Add a terminal screenshot showing the selected test data before the backup and restore drill.

![Output](screenshots/Data-Bf.png)

---

#### Screenshot 25 — Successful Backup Creation

Add a terminal screenshot showing successful backup creation and the backup file stored in the host backup directory.

![Output](screenshots/Backup.png)

---

#### Screenshot 26 — Controlled Data-Loss Test

Add a terminal screenshot showing that the selected test record was removed during the controlled data-loss test.

![Output](screenshots/Deleted.png)

---

#### Screenshot 27 — Restore Verification

Add a terminal screenshot showing successful restore and verification that the deleted test record is available again.

![Output](screenshots/Restored.png)

---

#### Screenshot 28 — Persistence After Down/Up Cycle

Add a terminal screenshot showing that database data remains available after a non-destructive Docker Compose down/up cycle.

Do not use `docker compose down -v`.

![Output](screenshots/Down-up2.png)

---

#### Screenshot 29 — Persistence and Backup Document

Add a screenshot showing the contents of:

```text
docs/05-persistence-and-backup.md
```

Include the backup plan and restore procedure.

![Output](screenshots/Docs-05.png)

---

# Task 6 — Configure Logging and Observability

## Goal

Configure useful reverse-proxy and backend logs without exposing sensitive information.

### Evidence

#### Screenshot 30 — Logging Configuration

Add a screenshot showing:

- Configuration for the selected reverse proxy
- Proxy log format
- Docker Compose host log-directory bind mount

![Output](screenshots/log.png)

---

#### Screenshot 31 — Persistent Proxy Logs and Backend Logs

Add a terminal screenshot showing:

- Selected reverse-proxy logs available from the host directory after a proxy restart
- Backend logs displayed through Docker Compose

![Output](screenshots/Persistent-log.png)

---

### Notes

Write a short note covering:

- The selected reverse proxy
- Host path used for reverse-proxy logs
- How backend logs are viewed
- Whether JSON or standard text logs were used
- Why passwords, tokens, headers, and database connection strings must not appear in logs

Write your note here.
- The selected reverse proxy: Nginx (nginx:1.28-alpine), configured in proxy/default.conf. It’s the only container that publishes a host port (80).
- Host path used for reverse-proxy logs: ~/theepicbook/logs/proxy/ on the VM, which holds access.log and error.log. This folder is bind-mounted to /var/log/nginx inside the container, so the logs are kept on the host and survive a proxy restart.
- How backend logs are viewed: The backend writes to stdout and stderr, and the logs are viewed with docker compose logs backend (for example, docker compose logs --tail=50 backend). Docker rotates them at 10 MB, keeping 3 files.
- Whether JSON or standard text logs were used: The access log uses structured JSON (Nginx log_format with escape=json), with one line per request recording the time, client IP, method, path, status, bytes, response time and which upstream served it. The error log uses Nginx’s standard text format. The backend also logs one JSON line per request.
- Why passwords, tokens, headers, and database connection strings must not appear in logs: Logs are read by more people and tools than the application itself, they’re kept for a long time, and they’re often copied to other systems. Any password, token, Authorization header, session cookie or connection string written to a log becomes a working credential for anyone who can read it. So the access log records the path without query strings and leaves out cookies and Authorization headers, and the backend logs only error names, never connection details.

---

# Task 7 — Deploy and Verify the Stack on a Cloud VM

## Goal

Deploy the completed Docker Compose stack on an AWS or Azure VM and verify public access.

### Evidence

#### Screenshot 32 — VM Public IP and Inbound Rules

Add a cloud-console screenshot showing:

- VM public IP address
- SSH port 22 restricted to your IP address
- HTTP port 80 allowed from Anywhere

![Output](screenshots/Instance.png)
![Output](screenshots/Security-group.png)

---

#### Screenshot 33 — Cloud VM Stack Verification

Add a VM terminal screenshot showing:

- Docker Compose service status
- Successful public health or API response
- No published database, frontend, or backend ports

![Output](screenshots/Verification.png)

---

#### Screenshot 34 — EpicBook Application on Cloud VM

Add a browser screenshot showing the EpicBook application loaded through the VM public IP address.

Add your full name as a clear caption below the screenshot.

![Output](screenshots/Epicbook-browser2.png)

---

### Notes

Write a short note covering:

- Cloud provider used
- VM operating system
- Public port exposed
- Security rules configured
- Confirmation that the application and backend API worked through the reverse proxy

Write your note here.
- Cloud provider used: Amazon Web Services (AWS), using an EC2 instance in the eu-north-1 (Stockholm) region.
- VM operating system: Ubuntu 26.04 LTS (64-bit), with Docker Engine and Docker Compose v2.
- Public port exposed: Port 80 (HTTP) only. It’s published by the Nginx reverse-proxy container, and the application is reached at http://13.60.196.64. The frontend (80), backend (8080) and MySQL (3306) container ports are not published to the host.
- Security rules configured: The EC2 security group allows SSH (port 22) from my IP address only and HTTP (port 80) from anywhere (0.0.0.0/0). There are no inbound rules for 3306 or 8080, so the database and backend can’t be reached from the internet.
- Confirmation that the application and backend API worked through the reverse proxy: Confirmed. The EpicBook home page loaded book data from MySQL at http://13.60.196.64, and GET /health returned 200 OK through Nginx.

---

# Task 8 — Automate Deployment with CI/CD (Optional)

## Goal

Optionally automate image build, image push, and deployment through GitHub Actions or Azure Pipelines.

### Optional Evidence

#### Optional Screenshot — Successful CI/CD Pipeline Run

Add a screenshot showing a successful pipeline run with build, image push, deployment, and verification stages.

![Output](screenshots/Deploy.png)
![Output](screenshots/Run.png)

---

### Optional Notes

Write a short note covering:

- CI/CD platform used
- Image-tagging method
- Registry used
- Deployment trigger
- Manual approval or secret-handling approach

Write your note here.
- CI/CD platform used: GitHub Actions, with the workflow file at .github/workflows/deploy.yml. Run #2 (commit 8ed2317) completed successfully, covering the build, image push, deployment and health-check steps.
- Image-tagging method: each image is tagged with the full Git commit SHA (github.sha). Every deployment maps to an exact commit, tags are never reused, and an older version can be redeployed by its SHA.
- Registry used: GitHub Container Registry (GHCR), as ghcr.io/iam-ginika/epicbook-backend and ghcr.io/iam-ginika/epicbook-frontend.
- Deployment trigger: a push to the main branch. The pipeline builds both images, pushes them to GHCR, connects to the VM over SSH, runs docker compose pull and docker compose up -d --wait with the new tags, then verifies http://13.60.196.64/health with curl --fail.
- Manual approval or secret-handling approach: there’s no manual approval step, because every push to main deploys automatically. The SSH private key, VM address, VM username and pinned SSH host keys are stored as GitHub Actions encrypted secrets, and registry access uses the job’s short-lived GITHUB_TOKEN. No passwords, keys or database values are in the pipeline file, and GitHub masks secrets as *** in the logs. SSH host-key verification stays on through the pinned known_hosts entry. Port 22 was opened to GitHub’s runners only for the deployment run, then restricted back to my IP.

---

# Task 9 — Perform Reliability Tests and Create an Operations Runbook

## Goal

Test controlled service failures and document safe operating procedures.

### Evidence

#### Screenshot 35 — Backend Failure and Recovery

Add a terminal screenshot showing:

- Backend failure test
- Expected unavailable response through the reverse proxy
- Backend restart
- Successful health-check recovery

![Output](screenshots/Backend-stopped-up.png)

---

#### Screenshot 36 — Database Failure and Recovery

Add a terminal screenshot showing:

- Database outage test
- Failed database-dependent request
- Database restart
- Successful application recovery

![Output](screenshots/Database-testing.png)

---

### Notes

Write a short operations runbook covering:

- Safe restart procedure for reverse proxy, frontend, backend, and database
- Backup and restore procedure
- Secret-rotation approach
- Database recovery procedure
- What to check when the application returns an error
- Results of backend and database reliability tests

# Operations Runbook – The EpicBook Application

## 1. Safe Restart Procedure

Restart services in a controlled order using Docker Compose:

```bash
docker compose restart database
docker compose restart backend
docker compose restart frontend
docker compose restart reverse-proxy
```

Check service health after restarting:

```bash
docker compose ps
```

For a full restart, use `docker compose up -d --wait`.

## 2. Backup and Restore Procedure

**Backup MySQL:**

```bash
docker compose exec database mysqldump -u root -p --all-databases > backup.sql
```

**Restore MySQL:**

```bash
docker compose exec -T database mysql -u root -p < backup.sql
```

Store backups securely and never commit them to Git.

## 3. Secret Rotation

- Update database passwords and application secrets.
- Update the relevant environment variables or secrets configuration.
- Restart affected services.
- Verify database connectivity and application functionality.
- Never expose secrets in source code or logs.

## 4. Database Recovery

- Check database status using `docker compose ps`.
- Inspect database logs using `docker compose logs database`.
- Restart the database if necessary.
- Verify MySQL health and backend connectivity.

## 5. Application Error Troubleshooting

- Check container status and health checks.
- Inspect backend and Nginx logs.
- Verify database connectivity.
- Check environment variables and network configuration.
- Test API and health endpoints using curl.

## 6. Reliability Test Results

**Backend failure test:** Stopped the backend and tested the health endpoint through Nginx. Restarted the backend and checked health recovery.

**Database failure test:** Stopped MySQL and tested the cart API. Restarted MySQL and verified database and application recovery.

Record the actual HTTP status codes and recovery results from the terminal tests as evidence.

---

# Final Public Application URL

**EpicBook URL:** http://13.60.196.64/

Replace the placeholder with your working public URL.

---

# GitHub Repository URL

**Your Fork or Repository URL:** https://github.com/iam-ginika/theepicbook

---

# LinkedIn Requirement

## Goal

Create a professional LinkedIn post of 6–10 lines about your EpicBook capstone deployment.

Your post must include:

- The architectural decision that most improved reliability
- Your biggest image-size reduction, with numbers
- Key production-hardening lessons
- A deployment verification image

### Evidence

**LinkedIn Post URL:** https://www.linkedin.com/posts/ginikandubuisi_devops-docker-dockercompose-share-7530991220752531456-jEPH/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD6T0TgBum59kWGrvQdH9mZyCcgf18-giQo

#### LinkedIn Post Screenshot

Add a screenshot of the published LinkedIn post showing the text body and deployment verification image.
![Output](screenshots/LinkedIn-7.png)

---

# Submission Checklist

- [ ] EpicBook repository reviewed and architecture diagram created
- [ ] Environment variables, ports, persistence, and health-check details documented
- [ ] Backend and frontend production Dockerfiles created
- [ ] Backend runs as a non-root user
- [ ] Docker image-size comparison completed
- [ ] Docker Compose stack includes reverse proxy, frontend, backend, and database
- [ ] `front-tier` and `back-tier` networks configured
- [ ] `db_data` named volume configured
- [ ] MySQL, backend, frontend, and reverse-proxy health checks configured
- [ ] Startup dependencies use `service_healthy`
- [ ] Nginx or Traefik selected as the only public reverse proxy
- [ ] Only reverse-proxy port 80 is publicly published
- [ ] Same-origin routing configured and CORS used only when required
- [ ] Backup, restore, and persistence testing completed
- [ ] Reverse-proxy and backend logs verified
- [ ] Cloud VM deployment verified through the public IP
- [ ] Backend and database reliability tests completed
- [ ] Screenshots 1–36 included
- [ ] Required notes completed
- [ ] LinkedIn post URL and screenshot included
- [ ] Full name visible in required screenshots or captions
- [ ] No passwords, tokens, private keys, account IDs, or other sensitive information exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) — Agentic AI Track.*
