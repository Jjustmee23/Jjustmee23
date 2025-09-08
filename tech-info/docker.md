# 🐳 Docker - Containerization Platform

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

## 📋 What is Docker?

**Docker** is a platform that uses OS-level virtualization to deliver software in packages called containers. Containers are lightweight, standalone, executable packages that include everything needed to run an application: code, runtime, system tools, system libraries, and settings.

## 🔧 Key Features

- **Containerization** - Lightweight virtualization technology
- **Portability** - "Build once, run anywhere"
- **Isolation** - Secure application separation
- **Scalability** - Easy horizontal scaling
- **Efficiency** - Minimal resource overhead
- **Version Control** - Image versioning and rollback

## 💡 Why I Use Docker

### 🏭 Industrial Applications
- **Mill Management Systems** - Consistent deployment across environments
- **Microservices Architecture** - Isolated service components
- **Development Environment** - Standardized development setup
- **CI/CD Pipelines** - Automated testing and deployment

### 🚀 DevOps Benefits
- **Environment Consistency** - Eliminates "works on my machine" issues
- **Resource Efficiency** - Better resource utilization than VMs
- **Rapid Deployment** - Quick application startup and scaling
- **Easy Maintenance** - Simplified updates and rollbacks

## 🛠️ My Docker Architecture

### Mill Management System Stack:
```yaml
# docker-compose.yml for complete mill system
version: '3.8'

services:
  # Web Application
  django-app:
    build: 
      context: ./backend
      dockerfile: Dockerfile
    container_name: mill_django
    ports:
      - "8000:8000"
    environment:
      - DEBUG=False
      - DATABASE_URL=postgresql://mill_user:${DB_PASSWORD}@postgres:5432/mill_db
      - REDIS_URL=redis://redis:6379/0
    depends_on:
      - postgres
      - redis
    volumes:
      - ./logs:/app/logs
      - static_volume:/app/staticfiles
    networks:
      - mill_network
    restart: unless-stopped

  # Database
  postgres:
    image: postgres:15-alpine
    container_name: mill_postgres
    environment:
      POSTGRES_DB: mill_db
      POSTGRES_USER: mill_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - mill_network
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: mill_redis
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - mill_network
    restart: unless-stopped

  # MQTT Broker
  mosquitto:
    image: eclipse-mosquitto:2
    container_name: mill_mqtt
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    networks:
      - mill_network
    restart: unless-stopped

  # Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: mill_nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/ssl:/etc/nginx/ssl
      - static_volume:/var/www/static
    depends_on:
      - django-app
    networks:
      - mill_network
    restart: unless-stopped

  # Monitoring
  prometheus:
    image: prom/prometheus:latest
    container_name: mill_prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - mill_network
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: mill_grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
    networks:
      - mill_network
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:
  static_volume:

networks:
  mill_network:
    driver: bridge
```

## 🌟 My Docker Projects

### 1. 🏭 Django Mill Application Dockerfile
```dockerfile
# Multi-stage build for production efficiency
FROM python:3.11-slim as builder

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Create virtual environment
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python dependencies
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# Production stage
FROM python:3.11-slim as production

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH"

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    libpq5 \
    && rm -rf /var/lib/apt/lists/*

# Copy virtual environment from builder stage
COPY --from=builder /opt/venv /opt/venv

# Create app user
RUN groupadd -r app && useradd -r -g app app

# Set work directory
WORKDIR /app

# Copy project
COPY . .

# Change ownership to app user
RUN chown -R app:app /app

# Switch to app user
USER app

# Collect static files
RUN python manage.py collectstatic --noinput

# Health check
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD python manage.py check --database default || exit 1

# Run application
EXPOSE 8000
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "mill_project.wsgi:application"]
```

### 2. 🤖 IoT Sensor Data Collector
```dockerfile
# Lightweight Python container for IoT data collection
FROM python:3.11-alpine

# Install system dependencies
RUN apk add --no-cache \
    gcc \
    musl-dev \
    linux-headers

# Set work directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY src/ .

# Create non-root user
RUN adduser -D -s /bin/sh sensor_user
USER sensor_user

# Environment variables
ENV MQTT_BROKER=mosquitto \
    MQTT_PORT=1883 \
    DATABASE_URL=postgresql://user:pass@postgres:5432/sensors

# Health check
HEALTHCHECK --interval=60s --timeout=10s --start-period=30s --retries=3 \
    CMD python health_check.py || exit 1

# Run the sensor collector
CMD ["python", "sensor_collector.py"]
```

### 3. 🔧 Development Environment
```dockerfile
# Development Dockerfile with hot reload
FROM python:3.11-slim

# Install system dependencies including development tools
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    git \
    vim \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Set work directory
WORKDIR /app

# Install Python dependencies
COPY requirements-dev.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements-dev.txt

# Install development tools
RUN pip install \
    black \
    flake8 \
    pytest \
    pytest-django \
    ipdb

# Copy project files
COPY . .

# Set environment for development
ENV DJANGO_SETTINGS_MODULE=mill_project.settings.development \
    DEBUG=True

# Expose port for development server
EXPOSE 8000

# Development command with auto-reload
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## 📊 Container Orchestration

### Docker Swarm Configuration:
```yaml
# docker-stack.yml for production deployment
version: '3.8'

services:
  django-app:
    image: mill-management:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
    networks:
      - mill_network
    secrets:
      - db_password
      - secret_key

  postgres:
    image: postgres:15-alpine
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - mill_network
    secrets:
      - db_password

secrets:
  db_password:
    external: true
  secret_key:
    external: true

networks:
  mill_network:
    driver: overlay
    attachable: true

volumes:
  postgres_data:
    driver: local
```

### Kubernetes Deployment:
```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mill-django-app
  labels:
    app: mill-django
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mill-django
  template:
    metadata:
      labels:
        app: mill-django
    spec:
      containers:
      - name: django
        image: mill-management:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: mill-secrets
              key: database-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready/
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 🔒 Security Best Practices

### Secure Dockerfile Practices:
```dockerfile
# Security-focused Dockerfile
FROM python:3.11-slim as base

# Create non-root user first
RUN groupadd -r app && useradd -r -g app app

# Install security updates
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    libpq5 && \
    rm -rf /var/lib/apt/lists/*

# Set secure permissions
WORKDIR /app
COPY --chown=app:app . .

# Remove unnecessary files
RUN rm -rf \
    .git \
    .pytest_cache \
    __pycache__ \
    *.pyc \
    tests/

# Switch to non-root user
USER app

# Use specific version tags
FROM python:3.11.5-slim

# Scan for vulnerabilities
RUN python -m pip install --upgrade pip safety && \
    safety check

# Set read-only root filesystem
VOLUME ["/tmp"]
```

### Container Security Scanning:
```bash
# Security scanning pipeline
#!/bin/bash

# Build image
docker build -t mill-management:latest .

# Scan for vulnerabilities
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $HOME/.cache:/tmp/.cache \
  aquasec/trivy image mill-management:latest

# Check for secrets
docker run --rm -v $(pwd):/src \
  zricethezav/gitleaks:latest detect --source="/src" --verbose

# Lint Dockerfile
docker run --rm -i hadolint/hadolint < Dockerfile
```

## 📈 Monitoring & Logging

### Container Monitoring:
```yaml
# Monitoring stack
version: '3.8'
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.ignored-mount-points'
      - '^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)'
    restart: unless-stopped
```

### Centralized Logging:
```yaml
# ELK Stack for log aggregation
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  elasticsearch_data:
```

## 🚀 CI/CD Integration

### GitHub Actions Docker Workflow:
```yaml
# .github/workflows/docker.yml
name: Docker Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          jjustmee23/mill-management:latest
          jjustmee23/mill-management:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
    
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        echo "${{ secrets.DEPLOY_KEY }}" | base64 -d > deploy_key
        chmod 600 deploy_key
        ssh -i deploy_key -o StrictHostKeyChecking=no user@production-server \
          "docker-compose pull && docker-compose up -d"
```

## 📚 Learning Resources

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)

## 🎯 Performance Optimization

### Image Size Optimization:
```dockerfile
# Multi-stage build to reduce image size
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY . .
RUN npm run build && \
    npm prune --production

# Final minimal image
FROM node:18-alpine
WORKDIR /app
COPY --from=production /app/dist ./dist
COPY --from=production /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

### Resource Management:
```yaml
# Resource limits and requests
services:
  django-app:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

---

*"Docker transforms the way we build, ship, and run applications - making deployment consistent and scalable."* - Danny Verheyen

[← Back to Main Profile](../README.md)
