# platform-bootstrap

End-to-end platform bootstrap and CI/CD setup using Ansible, Docker, Kubernetes, Helm, and Jenkins.

This project demonstrates how I bootstrapped a fresh Ubuntu server using Ansible
and deployed a containerised application on Kubernetes using Helm and Jenkins.

## Part 1: Infrastructure Bootstrap 

- Bootstrapped a headless Ubuntu server using Ansible
- Updated system packages
- Disabled swap for Kubernetes
- Configured UFW firewall with minimal required ports
- Installed Docker and enabled non-root access
- Configured a non-root user with:
  - SSH key-based access
  - Passwordless sudo privileges
- Deployed a test NGINX container to validate Docker setup

## Part 2: Docker — Containerization 

- Built a lightweight Flask-based web application
- Implemented a `/health` endpoint for container health checks
- Created a multi-stage Dockerfile to optimise image size
- Added `.dockerignore` to reduce build context
- Successfully built and ran the container locally

