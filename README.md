# platform-bootstrap

Bootstrap and CI/CD setup for a single-node Kubernetes environment.

This project demonstrates how I bootstrapped a fresh Ubuntu server using Ansible
and deploy a containerised application on Kubernetes using Helm and Jenkins.

## Part 1: Infrastructure Bootstrap (Completed)

- Bootstrapped a headless Ubuntu server using Ansible
- Updated system packages
- Disabled swap for Kubernetes
- Configured UFW firewall with minimal required ports
- Installed Docker and enabled non-root access
- Deployed a test NGINX container to validate Docker setup
