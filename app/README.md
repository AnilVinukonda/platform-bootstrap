# Application Container

This directory contains a lightweight Flask-based web application used to
demonstrate containerization and image optimization.

## Features
- Simple HTTP service
- `/health` endpoint for container health checks
- Designed for Kubernetes readiness

## Build Image
```bash
docker build -t platform-bootstrap-app .


## Run Container

docker run -p 8080:8080 platform-bootstrap-app


## Endpoints

/ – Application root

/health – Health check endpoint
