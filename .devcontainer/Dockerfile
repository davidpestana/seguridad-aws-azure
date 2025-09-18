# Imagen base oficial de DevContainers
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# Instala dependencias del sistema
RUN apt-get update && apt-get install -y \
    curl unzip gnupg lsb-release software-properties-common \
    python3 python3-pip \
    jq git bash \
    && rm -rf /var/lib/apt/lists/*

# --- Instalar AWS CLI v2 ---
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
 && unzip awscliv2.zip \
 && ./aws/install \
 && rm -rf awscliv2.zip aws

# --- Instalar Azure CLI ---
RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# --- Instalar Terraform (versión fija o última) ---
ENV TERRAFORM_VERSION=1.8.5
RUN curl -fsSL https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip \
    -o terraform.zip && \
    unzip terraform.zip && \
    mv terraform /usr/local/bin/ && \
    rm terraform.zip

# (Opcional) configura bash como shell por defecto
SHELL ["/bin/bash", "-c"]
