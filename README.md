# DSO_A3
# DSO101 - Assignment III: CI/CD Pipeline with GitHub Actions, Docker, and Render

**Student:** Sanjuck Subba  
**Course:** DSO101 - Continuous Integration and Continuous Deployment  
**Program:** Bachelor of Engineering in Software Engineering (SWE)  
**Submission Date:** 29th April  

---

## 🔗 Live Deployment Links

| Service | URL |
|---|---|
| **Backend API** | https://todo-backend-fvqt.onrender.com |
| **Frontend App** | https://todo-frontend-ihan.onrender.com |

---

## 📋 Project Overview

This assignment demonstrates a complete CI/CD pipeline that automatically:
1. Builds Docker containers for both the backend and frontend of a Todo List application
2. Pushes the containers to DockerHub
3. Deploys the containers on Render.com via deploy webhooks

---

## 🛠️ Tools & Technologies Used

| Tool | Purpose |
|---|---|
| GitHub | Hosting source code |
| GitHub Actions | CI/CD automation |
| Docker | Containerization |
| DockerHub | Container registry |
| Render.com | Cloud deployment |
| Node.js & Express | Backend runtime |
| React | Frontend framework |
| PostgreSQL | Database |
| Jest | Testing framework |
| Nginx | Frontend web server |

---

## 📁 Repository Structure

```
DSO_A3/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions CI/CD pipeline
├── backend/
│   ├── Dockerfile              # Backend container configuration
│   ├── server.js               # Express API server
│   ├── package.json            # Backend dependencies and scripts
│   ├── sum.test.js             # Backend test file
│   ├── .env                    # Local environment variables
│   └── .env.production         # Production environment variables
├── frontend/
│   ├── Dockerfile              # Frontend multi-stage build
│   ├── src/
│   │   ├── App.js              # Main React component
│   │   └── App.test.js         # Frontend test file
│   ├── public/
│   ├── package.json            # Frontend dependencies
│   └── .dockerignore
├── docker-compose.yml          # Local development compose file
├── render.yaml                 # Render deployment configuration
└── README.md                   # This file
```

---

## 📝 Steps Taken

### Task 1 — GitHub Repository Setup
- Verified the existing Node.js Todo application from Assignment 1
- Ensured `package.json` had the correct scripts: `start`, `test`, and `build`
- Fixed the start script from `node index.js` to `node server.js` to match the actual filename
- Confirmed the repository is public on GitHub

### Task 2 — Dockerfile Configuration

**Backend Dockerfile:**
- Used `node:20-alpine` as the base image
- Set working directory to `/app`
- Copied and installed dependencies
- Added `RUN npm test` to run tests during build
- Exposed port 5000
- Used `CMD ["npm", "start"]` to start the server

**Frontend Dockerfile:**
- Used a multi-stage build approach
- Stage 1: Build the React application using `node:18-alpine`
- Added `RUN CI=true npm test` to run tests without watch mode
- Stage 2: Serve the built files using `nginx:alpine`
- Exposed port 80

Both images were tested locally before pushing to DockerHub:
```bash
docker build -t sanjucksubba/todo-backend:latest ./backend
docker build -t sanjucksubba/todo-frontend:latest ./frontend
docker push sanjucksubba/todo-backend:latest
docker push sanjucksubba/todo-frontend:latest
```

### Task 3 — GitHub Actions Workflow
- Created `.github/workflows/deploy.yml`
- Configured the workflow to trigger on every push to the `main` branch
- Added 5 steps: Checkout, DockerHub Login, Build & Push Backend, Build & Push Frontend, Trigger Render Deployments
- Added GitHub Secrets for secure credential management:
  - `DOCKERHUB_USERNAME`
  - `DOCKERHUB_TOKEN`
  - `RENDER_DEPLOY_HOOK_BACKEND`
  - `RENDER_DEPLOY_HOOK_FRONTEND`

### Task 4 — Render.com Deployment
- Created two Web Services on Render.com using existing DockerHub images
- Backend service: deployed from `docker.io/sanjucksubba/todo-backend:latest`
- Frontend service: deployed from `docker.io/sanjucksubba/todo-frontend:latest`
- Configured environment variables for the backend PostgreSQL database connection
- Used the External Database URL for the Render PostgreSQL instance
- Obtained Deploy Hook URLs from each service's Settings and added them as GitHub Secrets

---

## ⚠️ Challenges Faced

### 1. Frontend Test Failing During Docker Build
The default Create React App test (`App.test.js`) was looking for "learn react" text, but the app had been changed to a Todo List. The Docker build failed because `RUN CI=true npm test` ran this outdated test.

**Fix:** Updated `App.test.js` to test for the actual "To-Do List" heading in the app.

### 2. Docker Push Authorization Error
Received `push access denied` error when trying to push images to DockerHub.

**Fix:** Ran `docker logout` followed by `docker login -u sanjucksubba` and re-authenticated using a newly generated Personal Access Token from DockerHub.

### 3. Database Connection Failed on Render
The backend was crashing with `getaddrinfo ENOTFOUND dpg-d7uppo9j2pic73c4vgfg-a` because the Internal Database URL was being used, which only works within Render's internal network.

**Fix:** Replaced the `DATABASE_URL` environment variable with the External Database URL (`dpg-d7uppo9j2pic73c4vgfg-a.oregon-postgres.render.com`) and added `?sslmode=require` at the end.

### 4. Render Deploy Hook Not Available Before Service Creation
The `RENDER_DEPLOY_HOOK` GitHub Secrets could not be added until the Render services were created and deployed first.

**Fix:** Manually pushed Docker images to DockerHub first, then created Render services, and finally added the deploy hook URLs as GitHub Secrets.

---

## 🎓 Learning Outcomes

- Understood how to containerize both frontend and backend applications using Docker
- Learned how to write multi-stage Dockerfiles to optimize image size
- Gained hands-on experience with GitHub Actions for automating CI/CD pipelines
- Learned how to securely manage credentials using GitHub Secrets instead of hardcoding them
- Understood the difference between Internal and External database URLs in cloud environments
- Learned how to use Render.com deploy webhooks to trigger automatic redeployments when new Docker images are pushed
- Understood the full CI/CD flow: code push → automated build → test → push to registry → deploy to cloud

---

## 📸 Screenshots

### 1. Successful GitHub Actions Workflow
![GitHub Actions Success](screenshots/github-actions.png)

### 2. DockerHub Images Pushed
![DockerHub](screenshots/dockerhub.png)

### 3. Render.com Backend Deployment
![Render Backend](screenshots/render-backend.png)

### 4. Render.com Frontend Deployment
![Render Frontend](screenshots/render-frontend.png)

---

## 🔧 GitHub Actions Workflow (deploy.yml)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: ["main"]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Backend Image
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/todo-backend:latest ./backend
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/todo-backend:latest

      - name: Build and Push Frontend Image
        run: |
          docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/todo-frontend:latest ./frontend
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/todo-frontend:latest

      - name: Trigger Render Deployment
        run: |
          curl -s "${{ secrets.RENDER_DEPLOY_HOOK_BACKEND }}"
          curl -s "${{ secrets.RENDER_DEPLOY_HOOK_FRONTEND }}"
```
