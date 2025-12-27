FROM jenkins/jenkins:lts-jdk17

USER root

# Install OS packages
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates \
    gnupg \
    lsb-release \
    apt-transport-https \
    && rm -rf /var/lib/apt/lists/*

# -----------------------
# Install Docker CLI
# -----------------------
RUN curl -fsSL https://get.docker.com | sh

# -----------------------
# Install kubectl
# -----------------------
RUN curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/kubectl

# -----------------------
# Install Helm
# -----------------------
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# -----------------------
# Permissions
# -----------------------
RUN usermod -aG docker jenkins

USER jenkins

