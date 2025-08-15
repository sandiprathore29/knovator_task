# Knovator Task – Full CI/CD Pipeline with Docker, Node.js, React & GitLab

This project demonstrates a complete CI/CD pipeline setup where both frontend (React) and backend (Node.js) applications are containerized, built, and deployed using Docker and GitLab CI/CD.
The images are pushed to GitLab Container Registry and deployed on a server with Docker Compose.

### Overview
This project demonstrates a complete DevOps workflow implementing:

- **Containerized Applications**: Multi-stage Docker builds for Node.js backend and React frontend
- **Reverse Proxy Setup**: Nginx-based routing and load balancing
- **CI/CD Automation**: GitLab pipeline with automated builds and deployments
- **Scalable Architecture**: Distributed Laravel application design
- **Security Best Practices**: Secure deployment strategies and configurations

### Architecture

```
knovator-task/
├── backend/
│   ├── Dockerfile            # Backend container configuration
│   ├── index.js             # Node.js entry point
│   ├── package.json         # Node.js dependencies
│   └── package-lock.json    # Lock file
├── frontend/
│   ├── Dockerfile           # Frontend container configuration
│   ├── nginx.conf           # Nginx configuration
│   ├── package.json         # React dependencies
│   ├── src/                 # React source files
│   └── public/              # Static assets
└── docker-compose.yml       # Multi-container orchestration
```

### Tech Stack
- **Backend:** Node.js, Express.js
- **Frontend:** React.js
- **Containerization:** Docker, Docker Compose
- **CI/CD:** GitLab CI/CD
- **Image Registry:** GitLab Container Registry
- **Deployment:** Ubuntu Server with Docker
- **Operating System**: Ubuntu 20.04 LTS


### Step 1: Application Containerization 
### Server Setup on AWS 
<img width="1093" height="177" alt="image" src="https://github.com/user-attachments/assets/71f70284-1a31-40c3-9c56-4cf9338c9674" />

#### Frontend (React)
**Multi-stage Dockerfile optimizations:**
- **Build Stage**: Uses Node.js 16 Alpine for compilation
- **Production Stage**: Lightweight Nginx Alpine for serving
- **Benefits**: smaller image size, enhanced security

**Frontend Dockerfile**
```dockerfile
# Build stage
FROM node:16-alpine AS build
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install --legacy-peer-deps
COPY frontend/. .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY frontend/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
<img width="1090" height="508" alt="image" src="https://github.com/user-attachments/assets/bbc5362d-5287-4110-81be-a77d8b1290b1" />

#### Backend (Node.js)

**Production-optimized container:**
- Uses Alpine Linux for minimal footprint
- Layer caching for faster builds
- Production-only dependencies

**Backend Dockerfile**
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY backend/package*.json ./
RUN npm install --only=production
COPY backend/. .
EXPOSE 3000
CMD ["npm", "start"]
```
<img width="1093" height="138" alt="image" src="https://github.com/user-attachments/assets/9f739abd-ed99-4421-b407-66678b33e39d" />

#### Nginx Reverse Proxy Configuration

**Features:**
- Single entry point (Port 8080)
- API request routing to backend
- Static file serving for frontend
- Load balancing ready

```nginx
server {
    listen 80;
    
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://backend:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
<img width="1093" height="138" alt="image" src="https://github.com/user-attachments/assets/04b20f44-e9e2-4c36-bf7a-e27c6db3b258" />


#### Docker Compose 

```yaml
version: "3"
services:
  frontend:
    image: registry.gitlab.com/sandiprathore29-group/knovator_task/frontend:latest
    ports:
      - "80:80"
  backend:
    image: registry.gitlab.com/sandiprathore29-group/knovator_task/backend:latest
    ports:
      - "3000:3000"
```

### Step 2: CI/CD Pipeline With Gitlab

#### Pipeline Stages

1. **Build & Push**: Automated Docker image creation and registry upload when code pushed/ merge in main branch
2. **Deploy**: Deploy the latest images on the server

#### GitLab CI/CD Configuration

```yaml
stages:
  - build_and_push
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""   # required for docker:dind
  FRONTEND_IMAGE: registry.gitlab.com/sandiprathore29-group/knovator_task/frontend:latest
  BACKEND_IMAGE: registry.gitlab.com/sandiprathore29-group/knovator_task/backend:latest

# Build and push frontend
frontend:
  stage: build_and_push
  image: docker:24
  services:
    - docker:dind
  variables:
    DOCKER_HOST: "tcp://docker:2375"
  script:
    - docker info
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    - docker build -t $FRONTEND_IMAGE -f frontend/Dockerfile .
    - docker push $FRONTEND_IMAGE

# Build and push backend
backend:
  stage: build_and_push
  image: docker:24
  services:
    - docker:dind
  variables:
    DOCKER_HOST: "tcp://docker:2375"
  script:
    - docker info
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
    - docker build -t $BACKEND_IMAGE -f backend/Dockerfile .
    - docker push $BACKEND_IMAGE

# Deploy stage
deploy:
  stage: deploy
  image: docker:24-cli
  tags:
    - deploy
  before_script:
    - docker --version
    - docker compose version
    # Authenticate to GitLab registry
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  script:
    - echo "Pulling latest images..."
    - docker compose pull
    - echo "Restarting services..."
    - docker compose up -d
    - echo "Cleaning up unused images..."
    - docker image prune -f
  only:
    - main
```
<img width="1153" height="317" alt="image" src="https://github.com/user-attachments/assets/19721e17-c5a8-4db7-8d71-9bd434e1143c" />

#### Security Features

- **Registry Authentication**: Secure image storage
- **Environment Variables**: Sensitive data protection
- **Branch Protection**: Main branch deployment only
- **Manual Deployment**: Production safety gate


#### GitLab Runner Setup
```bash
# Install GitLab Runner
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner

# Register runner
sudo gitlab-runner register \
  --url "https://gitlab.com/" \
  --registration-token "token" \
  --executor "docker" \
  --description "knovator-runner" \
  --docker-image "alpine:latest"
```
<img width="1086" height="445" alt="image" src="https://github.com/user-attachments/assets/515a8e11-db5b-448d-bfb4-fb7bd2ca805b" />


