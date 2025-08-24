# gitlab-ci-file
CI/CD implemented for a company

# CI/CD Pipeline for Portal (GitLab CI)

This repository uses **GitLab CI/CD** to build, merge, and deploy the Dockerized Django **portal** application to **staging** and **production** environments.

---

## 📌 Pipeline Overview

The pipeline is organized into **three stages**:

1. **Build** – Build & push Docker images for staging or production.
2. **Merge** – (Manual) Merge `develop` → `main` during releases.
3. **Deploy** – Deploy the application to staging or production servers via SSH & Docker Compose.

---

## 🚀 Workflow

### ▶ Staging (develop branch)
- Triggered when code is pushed to the `develop` branch.
- Builds a Docker image:
- Deploys automatically to the **staging server** at `192.168.4.15`.

### 🏷 Production (tags)
- Triggered when a **Git tag** (release) is pushed.
- Builds a Docker image:
- (Manual step) Merges `develop` → `main` via `merge_to_main` job.
- Deploys automatically to the **production server** at `192.168.4.32`.

---

## 🔑 Required CI/CD Variables

Set these variables in GitLab **Project Settings → CI/CD → Variables**:

- `REGIS_URL` — Docker registry URL (e.g., `192.168.4.15:6000`)
- `SSH_PRIVATE_KEY_ENCODED` — Base64 encoded SSH private key
- `USER_NAME` — Git user name (for merge commits)
- `USER_EMAIL` — Git user email (for merge commits)

> ⚠️ Make sure the corresponding public key is added to:
> - `administrator@192.168.4.15` (staging server)
> - `milan@192.168.4.32` (production server)

---

## 📂 Job Details

### 🛠 Build Jobs
- **Staging (`develop`)**
- Builds image with tag `staging-$CI_COMMIT_SHORT_SHA`
- Pushes to `$REGIS_URL`
- **Production (`tags`)**
- Builds image with tag `$CI_COMMIT_TAG`
- Pushes to `$REGIS_URL`

### 🔀 Merge Job
- Runs on **tags** (manual trigger).
- Merges `develop` → `main` branch:
- Prefers **develop’s changes** in case of conflicts.
- Pushes result to `origin main`.

### 🚚 Deploy Jobs
- **Staging (`develop`)**
- SSH into `192.168.4.15` (`administrator`)
- Runs `docker compose pull` & `docker compose up -d --force-recreate`
- Cleans up old containers and images
- **Production (`tags`)**
- SSH into `192.168.4.32` (`milan`)
- Runs `docker compose pull` & `docker compose up -d --force-recreate`
- Cleans up old containers and images

---

## 📝 Usage Guide

### Staging Flow
1. Push to `develop`.
2. Pipeline builds & deploys automatically.
3. Verify changes on the staging environment.

### Production Release Flow
1. Checkout & pull latest `develop`.
2. Create a tag:
 ```bash
 git tag v1.2.3
 git push origin v1.2.3
