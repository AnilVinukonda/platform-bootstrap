# Ansible Bootstrap

This directory contains Ansible playbooks used to bootstrap a fresh Ubuntu
server for container orchestration and CI/CD.

## Responsibilities
- Update and upgrade system packages
- Disable swap as required by Kubernetes
- Configure UFW firewall with minimal required ports
- Install and configure Docker with non-root access
- Configure a non-root user with:
  - SSH key-based access
  - Passwordless sudo privileges
- Deploy a test NGINX container to validate Docker setup

## Usage
```bash
ansible-playbook -i inventory site.yml --ask-become-pass

