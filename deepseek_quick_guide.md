# DeepSeek Quick Installation Guide

This guide provides a streamlined approach to setting up DeepSeek models locally using Docker, Ollama, and OpenWebUI. The setup script automates the entire process from Docker installation to model deployment.

## Overview

The automation script handles:
- Docker installation with proper DNS configuration
- Ollama (LLM backend) deployment
- OpenWebUI (frontend interface) setup
- Model selection and installation

## System Requirements

Minimum requirements vary based on the model chosen:
- deepseek-r1:1.5b: 1.1GB disk space
- deepseek-r1:7b: 4.7GB disk space
- deepseek-r1:8b: 4.8GB disk space
- deepseek-r1:14b: 9.0GB disk space
- deepseek-r1:32b: 20GB disk space
- deepseek-r1:70b: 43GB disk space
- deepseek-r1:671b: 404GB disk space

## What the Script Does

1. **Docker Setup**
   - Installs Docker if not present
   - Configures DNS settings to prevent lookup issues
   - Sets up Docker daemon configuration

2. **Container Deployment**
   - Creates a Docker Compose configuration
   - Deploys Ollama container (LLM backend)
   - Deploys OpenWebUI container (user interface)
   - Sets up necessary volumes and networking

3. **Model Installation**
   - Presents available DeepSeek models with sizes
   - Handles model download and installation
   - Shows real-time download progress

## Container Architecture

The setup uses two main containers:
- `ollama`: Handles the LLM operations and model management
- `webui`: Provides the user interface and API interactions

## Post-Installation

After installation:
1. The WebUI will be available at `http://YOUR_IP:7000`
2. Create your admin account on first access
3. Start using DeepSeek through the web interface


## Optional commands

```bash
# Copy and paste this entire block into your terminal
sudo apt update && sudo apt upgrade -y 
apt-get install sudo -y
sudo apt install curl -y
sudo apt install net-tools -y
```


## Installation Command

```bash
# Copy and paste this entire block into your terminal
sudo mkdir -p /etc/docker
cat << 'EOF' | sudo tee /etc/docker/daemon.json
{
    "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF

curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

sudo mkdir -p /opt/deepseek
cat << 'EOF' | sudo tee /opt/deepseek/docker-compose.yaml
services:
  webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: webui
    ports:
      - 7000:8080/tcp
    volumes:
      - open-webui:/app/backend/data
    extra_hosts:
      - "host.docker.internal:host-gateway"
    depends_on:
      - ollama
    restart: unless-stopped

  ollama:
    image: ollama/ollama
    container_name: ollama
    ports:
      - 11434:11434/tcp
    volumes:
      - ollama:/root/.ollama
    restart: unless-stopped

volumes:
  ollama:
  open-webui:
EOF

cd /opt/deepseek
sudo docker compose up -d

IP=$(ip route get 1 | awk '{print $7;exit}')

echo "
Available DeepSeek models:
1) deepseek-r1:1.5b  (1.1GB disk)
2) deepseek-r1:7b    (4.7GB disk)
3) deepseek-r1:8b    (4.8GB disk)
4) deepseek-r1:14b   (9.0GB disk)
5) deepseek-r1:32b   (20GB disk)
6) deepseek-r1:70b   (43GB disk)
7) deepseek-r1:671b  (404GB disk)

Enter model number (1-7): "
read model_choice

case $model_choice in
    1) model="deepseek-r1:1.5b" ;;
    2) model="deepseek-r1:7b" ;;
    3) model="deepseek-r1:8b" ;;
    4) model="deepseek-r1:14b" ;;
    5) model="deepseek-r1:32b" ;;
    6) model="deepseek-r1:70b" ;;
    7) model="deepseek-r1:671b" ;;
    *) echo "Invalid choice"; exit 1 ;;
esac

echo "Pulling $model..."
sudo docker exec -it ollama ollama pull "$model"

echo "Model $model installed successfully!"
echo "Access WebUI at http://$IP:7000"
```

After installation, access the WebUI at `http://YOUR_IP:7000` and create your admin account.
