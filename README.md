# Knovator DevOps Task

A comprehensive DevOps implementation containerized application deployment with Gitlab CI/CD automation and scalable architecture design.


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

- **Operating System**: Ubuntu 20.04 LTS
- **Docker**: Version 20.10+
- **Docker Compose**: Version 2.0+
- **GitLab Runner**: Self-hosted VM-based runner
- **Git**: Version 2.25+

### Step 1: Application Containerization

#### Frontend (React)
**Multi-stage Dockerfile optimizations:**
- **Build Stage**: Uses Node.js 16 Alpine for compilation
- **Production Stage**: Lightweight Nginx Alpine for serving
- **Benefits**: smaller image size, enhanced security

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

#### Backend (Node.js)

**Production-optimized container:**
- Uses Alpine Linux for minimal footprint
- Layer caching for faster builds
- Production-only dependencies

```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY backend/package*.json ./
RUN npm install --only=production
COPY backend/. .
EXPOSE 3000
CMD ["npm", "start"]
```

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

### Step 2: CI/CD Pipeline With Gitlab

#### Pipeline Stages

1. **Build & Push**: Automated Docker image creation and registry upload
2. **Deploy**: Automated deployment to target environments

#### GitLab CI/CD Configuration

```yaml
stages:
  - build_and_push
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""
  FRONTEND_IMAGE: registry.gitlab.com/sandiprathore29-group/knovator_task/frontend:latest
  BACKEND_IMAGE: registry.gitlab.com/sandiprathore29-group/knovator_task/backend:latest

frontend:
  stage: build_and_push
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $FRONTEND_IMAGE -f frontend/Dockerfile .
    - docker push $FRONTEND_IMAGE
  only:
    - main

backend:
  stage: build_and_push
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $BACKEND_IMAGE -f backend/Dockerfile .
    - docker push $BACKEND_IMAGE
  only:
    - main

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production environment"
    - # Add deployment commands here
  only:
    - main
  when: manual
```

#### Security Features

- **Registry Authentication**: Secure image storage
- **Environment Variables**: Sensitive data protection
- **Branch Protection**: Main branch deployment only
- **Manual Deployment**: Production safety gate

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

**Gitlab configuration and CICD**
<img width="1338" height="541" alt="image" src="https://github.com/user-attachments/assets/2036677d-f613-4a3f-9c95-aa0cb8a12f1c" />
<img width="1338" height="541" alt="image" src="https://github.com/user-attachments/assets/ad02c8d3-805d-401b-823e-6586c1e6521a" />
<img width="1338" height="541" alt="image" src="https://github.com/user-attachments/assets/387be3f0-b577-4e34-b2f3-2a23f85c4647" />

**Access the application**
   - Frontend: http://localhost
   - Backend API: http://localhost:3000
   - Full application: http://localhost:8080
   - nginx proxy: http://localhost/api to http://localhost:3000
<img width="1358" height="556" alt="image" src="https://github.com/user-attachments/assets/eab89eee-e899-401d-b954-998afc2e1a60" />
<img width="1362" height="139" alt="image" src="https://github.com/user-attachments/assets/8c87f1e9-9aa1-422a-bbbc-f82d88bc52d1" />
<img width="1358" height="556" alt="image" src="https://github.com/user-attachments/assets/12d01481-6098-4990-b650-4a82fcee12b3" />

