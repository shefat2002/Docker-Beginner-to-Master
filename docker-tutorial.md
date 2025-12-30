# Docker Complete Tutorial: From Beginner to Master

## Table of Contents
- [What is Docker?](#what-is-docker)
- [Installation](#installation)
- [Docker Basics](#docker-basics)
- [Images and Containers](#images-and-containers)
- [Dockerfile](#dockerfile)
- [Docker Compose](#docker-compose)
- [Networking](#networking)
- [Volumes and Data Management](#volumes-and-data-management)
- [Best Practices](#best-practices)
- [Advanced Topics](#advanced-topics)
- [Troubleshooting](#troubleshooting)

## What is Docker?

Docker is a containerization platform that packages applications and their dependencies into lightweight, portable containers. These containers can run consistently across different environments.

### Key Benefits:
- **Consistency**: Same environment across development, testing, and production
- **Portability**: Run anywhere Docker is installed
- **Efficiency**: Lightweight compared to virtual machines
- **Scalability**: Easy horizontal scaling
- **Isolation**: Applications run in isolated environments

### Docker vs Virtual Machines:
```
Virtual Machines:
┌─────────────────────────────────────────┐
│           Application                   │
├─────────────────────────────────────────┤
│           Guest OS                      │
├─────────────────────────────────────────┤
│          Hypervisor                     │
├─────────────────────────────────────────┤
│           Host OS                       │
└─────────────────────────────────────────┘

Docker Containers:
┌─────────────────────────────────────────┐
│           Application                   │
├─────────────────────────────────────────┤
│         Docker Engine                   │
├─────────────────────────────────────────┤
│           Host OS                       │
└─────────────────────────────────────────┘
```

## Installation

### Windows
1. Download Docker Desktop from [docker.com](https://docker.com)
2. Run the installer
3. Restart your computer
4. Verify installation: `docker --version`

### macOS
1. Download Docker Desktop from [docker.com](https://docker.com)
2. Drag Docker.app to Applications folder
3. Launch Docker Desktop
4. Verify installation: `docker --version`

### Linux (Ubuntu)
```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Add Docker's GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker repository
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Install Docker
sudo apt update
sudo apt install docker-ce

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group (optional)
sudo usermod -aG docker $USER
```

## Docker Basics

### Essential Commands

#### System Information
```bash
# Check Docker version
docker --version

# Show system information
docker system info

# Show Docker disk usage
docker system df

# Clean up unused resources
docker system prune
```

#### Working with Images
```bash
# List local images
docker images
docker image ls

# Pull an image from registry
docker pull nginx
docker pull node:16-alpine

# Remove an image
docker rmi nginx
docker image rm nginx

# Search for images
docker search nginx

# Show image history
docker history nginx
```

#### Working with Containers
```bash
# Run a container
docker run nginx
docker run -d nginx                    # Detached mode
docker run -it ubuntu bash            # Interactive terminal
docker run -p 8080:80 nginx          # Port mapping
docker run --name my-nginx nginx     # Custom name

# List containers
docker ps                             # Running containers
docker ps -a                         # All containers

# Stop a container
docker stop container_id
docker stop container_name

# Start a stopped container
docker start container_id

# Restart a container
docker restart container_id

# Remove a container
docker rm container_id
docker rm -f container_id             # Force remove running container

# Execute commands in running container
docker exec -it container_id bash
docker exec container_id ls -la

# View container logs
docker logs container_id
docker logs -f container_id           # Follow logs
```

## Images and Containers

### Understanding Images
Images are read-only templates used to create containers. They contain:
- Application code
- Runtime environment
- System libraries
- Dependencies
- Configuration files

### Image Layers
Docker images are built in layers:
```
┌─────────────────┐ ← Application Layer
├─────────────────┤ ← Dependencies Layer  
├─────────────────┤ ← Runtime Layer
├─────────────────┤ ← OS Layer
└─────────────────┘ ← Base Layer
```

### Container Lifecycle
```bash
# Container states: Created → Running → Paused → Stopped → Deleted

# Create container without starting
docker create nginx

# Start created container
docker start container_id

# Pause/unpause container
docker pause container_id
docker unpause container_id

# Kill container (force stop)
docker kill container_id
```

### Practical Examples

#### Example 1: Web Server
```bash
# Run Nginx web server
docker run -d -p 8080:80 --name web-server nginx

# Test the server
curl http://localhost:8080

# View logs
docker logs web-server

# Stop and remove
docker stop web-server
docker rm web-server
```

#### Example 2: Database
```bash
# Run MySQL database
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  mysql:8.0

# Connect to database
docker exec -it mysql-db mysql -u root -p
```

#### Example 3: Development Environment
```bash
# Run Node.js development environment
docker run -it \
  --name node-dev \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:16-alpine \
  sh
```

## Dockerfile

A Dockerfile is a text file with instructions to build Docker images.

### Basic Dockerfile Structure
```dockerfile
# Use official base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Define startup command
CMD ["npm", "start"]
```

### Dockerfile Instructions

#### FROM
Specifies the base image:
```dockerfile
FROM ubuntu:20.04
FROM node:16-alpine
FROM python:3.9-slim
```

#### WORKDIR
Sets the working directory:
```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
```

#### COPY vs ADD
```dockerfile
# COPY - Simple file copying
COPY package.json ./
COPY src/ ./src/

# ADD - Advanced copying (supports URLs and archives)
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /opt/
```

#### RUN
Executes commands during build:
```dockerfile
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install -r requirements.txt

# Multi-line RUN
RUN apt-get update \
    && apt-get install -y \
        curl \
        git \
        vim \
    && rm -rf /var/lib/apt/lists/*
```

#### ENV
Sets environment variables:
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgresql://user:pass@db:5432/mydb
```

#### EXPOSE
Documents which ports the container listens on:
```dockerfile
EXPOSE 3000
EXPOSE 80 443
```

#### CMD vs ENTRYPOINT
```dockerfile
# CMD - Default command (can be overridden)
CMD ["npm", "start"]
CMD npm start
CMD ["/bin/bash"]

# ENTRYPOINT - Always executed (cannot be overridden)
ENTRYPOINT ["python", "app.py"]
ENTRYPOINT ["/docker-entrypoint.sh"]

# Combined usage
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "3000"]
```

### Real-World Examples

#### Node.js Application
```dockerfile
FROM node:16-alpine

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
COPY package*.json ./
RUN npm ci --only=production

# Bundle app source
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

EXPOSE 3000

CMD ["node", "server.js"]
```

#### Python Application
```dockerfile
FROM python:3.9-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /code

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt /code/
RUN pip install -r requirements.txt

# Copy project
COPY . /code/

# Create user
RUN adduser --disabled-password --gecos '' appuser
USER appuser

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "myapp.wsgi:application"]
```

#### Multi-stage Build
```dockerfile
# Build stage
FROM node:16-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built assets from builder stage
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Building Images
```bash
# Build image from Dockerfile
docker build -t my-app .
docker build -t my-app:v1.0 .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t my-app .

# Build from specific Dockerfile
docker build -f Dockerfile.prod -t my-app .

# Build without cache
docker build --no-cache -t my-app .
```

## Docker Compose

Docker Compose manages multi-container applications using YAML files.

### Basic docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
    depends_on:
      - db
    volumes:
      - .:/app
      - /app/node_modules

  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Docker Compose Commands
```bash
# Start services
docker-compose up
docker-compose up -d              # Detached mode
docker-compose up --build         # Rebuild images

# Stop services
docker-compose down
docker-compose down -v            # Remove volumes
docker-compose down --rmi all     # Remove images

# View running services
docker-compose ps

# View logs
docker-compose logs
docker-compose logs web           # Specific service
docker-compose logs -f            # Follow logs

# Execute commands
docker-compose exec web bash
docker-compose exec db psql -U user myapp

# Scale services
docker-compose up --scale web=3

# Restart services
docker-compose restart
docker-compose restart web
```

### Advanced Compose Features

#### Environment Variables
```yaml
# .env file
DATABASE_URL=postgresql://user:pass@db:5432/mydb
REDIS_URL=redis://redis:6379

# docker-compose.yml
version: '3.8'
services:
  web:
    build: .
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL
    env_file:
      - .env
```

#### Networks
```yaml
version: '3.8'

services:
  web:
    build: .
    networks:
      - frontend
      - backend

  db:
    image: postgres:13
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

#### Health Checks
```yaml
version: '3.8'

services:
  web:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Real-World Example: Full Stack Application
```yaml
version: '3.8'

services:
  # Frontend (React)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://localhost:8000
    depends_on:
      - backend

  # Backend (Node.js)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  # Database
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # Cache
  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - frontend
      - backend

volumes:
  postgres_data:
  redis_data:
```

## Networking

### Network Types

#### Bridge (Default)
```bash
# Create custom bridge network
docker network create my-network

# Run containers on custom network
docker run -d --name web1 --network my-network nginx
docker run -d --name web2 --network my-network nginx

# Containers can communicate by name
docker exec web1 ping web2
```

#### Host
```bash
# Use host network (no network isolation)
docker run --network host nginx
```

#### None
```bash
# No networking
docker run --network none alpine
```

### Network Commands
```bash
# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Create network
docker network create --driver bridge my-net
docker network create --subnet=172.20.0.0/16 my-net

# Connect container to network
docker network connect my-net container_name

# Disconnect container from network
docker network disconnect my-net container_name

# Remove network
docker network rm my-net
```

### Port Management
```bash
# Port mapping examples
docker run -p 8080:80 nginx              # Host:Container
docker run -p 127.0.0.1:8080:80 nginx   # Bind to specific IP
docker run -p 8080-8090:8080-8090 nginx # Port range
docker run -P nginx                      # Auto-assign ports

# Check port mappings
docker port container_name
```

### Service Discovery
```yaml
# In docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    environment:
      # Use service names for internal communication
      - DATABASE_HOST=database
      - CACHE_HOST=cache
    depends_on:
      - database
      - cache

  database:
    image: postgres:13
    # Accessible as 'database' hostname

  cache:
    image: redis:alpine
    # Accessible as 'cache' hostname
```

## Volumes and Data Management

### Volume Types

#### Named Volumes
```bash
# Create named volume
docker volume create my-data

# Use named volume
docker run -v my-data:/data alpine

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data

# Remove volume
docker volume rm my-data
```

#### Bind Mounts
```bash
# Mount host directory
docker run -v /host/path:/container/path alpine
docker run -v $(pwd):/app node:16-alpine

# Read-only mount
docker run -v /host/path:/container/path:ro alpine
```

#### tmpfs Mounts (Linux only)
```bash
# Mount tmpfs (in-memory)
docker run --tmpfs /tmp alpine
```

### Volume Examples

#### Development Setup
```bash
# Mount source code for live development
docker run -it \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:16-alpine \
  npm run dev
```

#### Database Persistence
```bash
# PostgreSQL with persistent data
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mypassword \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:13
```

#### Configuration Files
```bash
# Mount configuration
docker run -d \
  --name nginx \
  -v /host/nginx.conf:/etc/nginx/nginx.conf:ro \
  -p 80:80 \
  nginx
```

### Backup and Restore

#### Backup Volume
```bash
# Backup named volume
docker run --rm \
  -v my-data:/data \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/backup.tar.gz -C /data .
```

#### Restore Volume
```bash
# Restore to named volume
docker run --rm \
  -v my-data:/data \
  -v $(pwd):/backup \
  alpine \
  sh -c "cd /data && tar xzf /backup/backup.tar.gz"
```

#### Database Backup
```bash
# Backup PostgreSQL
docker exec postgres pg_dump -U user database > backup.sql

# Restore PostgreSQL
docker exec -i postgres psql -U user database < backup.sql
```

## Best Practices

### Image Optimization

#### Multi-stage Builds
```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

#### Layer Optimization
```dockerfile
# Bad - creates multiple layers
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# Good - single layer
FROM ubuntu:20.04
RUN apt-get update \
    && apt-get install -y \
        curl \
        git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

#### Use .dockerignore
```dockerignore
# .dockerignore file
node_modules
npm-debug.log
.git
.gitignore
README.md
Dockerfile
docker-compose.yml
.env
coverage/
.nyc_output
```

### Security Best Practices

#### Use Official Images
```dockerfile
# Good - official images
FROM node:16-alpine
FROM postgres:13
FROM nginx:alpine

# Avoid - unknown sources
FROM random-user/suspicious-image
```

#### Non-root User
```dockerfile
FROM node:16-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Change ownership of app directory
WORKDIR /app
COPY --chown=nextjs:nodejs . .

# Switch to non-root user
USER nextjs

CMD ["node", "server.js"]
```

#### Secrets Management
```bash
# Use Docker secrets (Swarm mode)
echo "mypassword" | docker secret create db_password -

# Use external secret management
docker run -e DATABASE_URL="$(vault kv get -field=url secret/db)" myapp
```

#### Scan for Vulnerabilities
```bash
# Scan image for vulnerabilities
docker scan nginx:latest

# Use security scanning in CI/CD
docker build -t myapp .
docker scan myapp
```

### Performance Optimization

#### Resource Limits
```bash
# Set CPU and memory limits
docker run --cpus="1.5" --memory="512m" nginx

# In docker-compose.yml
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

#### Health Checks
```dockerfile
FROM nginx:alpine

# Add health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

EXPOSE 80
```

#### Logging
```bash
# Configure log driver
docker run --log-driver=json-file --log-opt max-size=10m nginx

# In docker-compose.yml
services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## Advanced Topics

### Docker Swarm

#### Initialize Swarm
```bash
# Initialize swarm mode
docker swarm init

# Join worker node
docker swarm join --token SWMTKN-1-... manager-ip:2377

# List nodes
docker node ls
```

#### Deploy Stack
```yaml
# docker-stack.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      update_config:
        parallelism: 1
        delay: 10s

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == manager]

secrets:
  db_password:
    external: true
```

```bash
# Deploy stack
docker stack deploy -c docker-stack.yml myapp

# List stacks
docker stack ls

# List services
docker service ls

# Scale service
docker service scale myapp_web=5
```

### Container Orchestration with Kubernetes

#### Convert Compose to Kubernetes
```bash
# Install kompose
curl -L https://github.com/kubernetes/kompose/releases/download/v1.26.1/kompose-linux-amd64 -o kompose
chmod +x kompose
sudo mv kompose /usr/local/bin

# Convert docker-compose.yml to Kubernetes manifests
kompose convert
```

### CI/CD Integration

#### GitHub Actions
```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1
    
    - name: Login to DockerHub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v2
      with:
        context: .
        push: true
        tags: |
          myapp:latest
          myapp:${{ github.sha }}
```

#### GitLab CI
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

build:
  stage: build
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

test:
  stage: test
  script:
    - docker run --rm $DOCKER_IMAGE npm test

deploy:
  stage: deploy
  script:
    - docker pull $DOCKER_IMAGE
    - docker-compose up -d
  only:
    - main
```

### Monitoring and Logging

#### Container Monitoring
```yaml
# monitoring/docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

volumes:
  prometheus_data:
  grafana_data:
```

#### Log Aggregation
```yaml
# logging/docker-compose.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

## Troubleshooting

### Common Issues and Solutions

#### Container Won't Start
```bash
# Check container logs
docker logs container_name

# Check container configuration
docker inspect container_name

# Run container in interactive mode
docker run -it image_name /bin/bash
```

#### Port Already in Use
```bash
# Find process using port
sudo lsof -i :8080
sudo netstat -tulpn | grep 8080

# Kill process
sudo kill -9 PID

# Use different port
docker run -p 8081:80 nginx
```

#### Permission Denied
```bash
# Fix file permissions
sudo chown -R $USER:$USER ./app
chmod -R 755 ./app

# Use correct user in Dockerfile
USER node
USER 1001
```

#### Out of Disk Space
```bash
# Clean up Docker resources
docker system prune -a
docker volume prune
docker image prune -a

# Remove all stopped containers
docker container prune

# Check disk usage
docker system df
```

#### Network Issues
```bash
# Check network configuration
docker network ls
docker network inspect bridge

# Test connectivity
docker exec container_name ping google.com
docker exec container_name nslookup database

# Restart Docker daemon
sudo systemctl restart docker
```

### Debugging Techniques

#### Interactive Debugging
```bash
# Override entrypoint for debugging
docker run -it --entrypoint /bin/bash image_name

# Debug running container
docker exec -it container_name /bin/bash

# Copy files from container
docker cp container_name:/path/to/file ./local/path

# Copy files to container
docker cp ./local/file container_name:/path/to/destination
```

#### Performance Debugging
```bash
# Monitor container resource usage
docker stats
docker stats container_name

# Check container processes
docker exec container_name ps aux
docker exec container_name top

# Monitor logs in real-time
docker logs -f container_name
```

#### Network Debugging
```bash
# Inspect container network
docker exec container_name ip addr show
docker exec container_name netstat -tulpn

# Test service connectivity
docker run --rm -it --network container_network alpine sh
# Inside container:
ping service_name
wget -qO- http://service_name:port/health
```

### Best Practices for Debugging

1. **Use Layered Logging**:
   ```dockerfile
   # Add debug information
   RUN echo "Installing dependencies..." && \
       npm install && \
       echo "Dependencies installed successfully"
   ```

2. **Health Checks**:
   ```dockerfile
   HEALTHCHECK --interval=30s --timeout=3s \
     CMD curl -f http://localhost:3000/health || exit 1
   ```

3. **Graceful Shutdown**:
   ```dockerfile
   # Handle SIGTERM properly
   STOPSIGNAL SIGTERM
   ```

4. **Environment-specific Configuration**:
   ```bash
   # Use different configs for different environments
   docker run -e NODE_ENV=development myapp
   docker run -e NODE_ENV=production myapp
   ```

### Performance Optimization Tips

1. **Use Alpine Images**:
   ```dockerfile
   FROM node:16-alpine  # Instead of node:16
   FROM python:3.9-alpine  # Instead of python:3.9
   ```

2. **Minimize Layers**:
   ```dockerfile
   # Combine RUN commands
   RUN apt-get update && \
       apt-get install -y package1 package2 && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

3. **Use Multi-stage Builds**:
   ```dockerfile
   FROM node:16 AS build
   # Build steps...
   
   FROM node:16-alpine AS production
   COPY --from=build /app/dist ./dist
   ```

4. **Optimize Docker Context**:
   ```dockerignore
   node_modules
   .git
   *.md
   tests/
   ```

## Conclusion

Docker is a powerful platform that revolutionizes how we develop, deploy, and manage applications. This tutorial covered:

- **Fundamentals**: Understanding containers, images, and Docker architecture
- **Practical Skills**: Building images, running containers, managing data
- **Advanced Topics**: Orchestration, monitoring, security, and optimization
- **Best Practices**: Security, performance, and maintainability guidelines

### Next Steps

1. **Practice Projects**:
   - Containerize your existing applications
   - Build a multi-service application with Docker Compose
   - Set up CI/CD pipelines with Docker

2. **Advanced Learning**:
   - Kubernetes orchestration
   - Docker Swarm clustering
   - Advanced networking concepts
   - Security hardening techniques

3. **Production Deployment**:
   - Cloud provider container services (ECS, GKE, AKS)
   - Container monitoring and logging
   - Performance optimization at scale

### Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)

Remember: The best way to learn Docker is by practicing. Start with simple examples and gradually work your way up to more complex scenarios. Happy containerizing!