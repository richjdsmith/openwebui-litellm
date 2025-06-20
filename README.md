# OpenWebUI + LiteLLM Stack

This project provides a ready-to-use Docker-based setup for deploying OpenWebUI with LiteLLM as an AI API gateway. It's designed for easy deployment and management, even for users with limited Docker or system administration experience.

## Overview

This stack includes:
*   **OpenWebUI:** A user-friendly and feature-rich web interface for interacting with various large language models.
*   **LiteLLM:** A powerful API gateway that standardizes API calls to over 100 LLM providers, including OpenAI, Anthropic, Gemini, and more. It provides a unified interface for model management, routing, and cost tracking.
*   **PostgreSQL:** A dedicated database for LiteLLM to store logs, virtual keys, and other persistent data.
*   **Watchtower:** A container that automatically updates the running services to their latest versions.

This setup is ideal for teams or individuals looking to create a centralized, self-hosted AI interaction platform.

## Prerequisites

Before you begin, ensure you have the following installed on your server (these instructions assume an Ubuntu/Debian-based system):

*   A server running a recent version of Ubuntu (e.g., 22.04 LTS).
*   Docker
*   Docker Compose

### 1. Install Docker and Docker Compose

If you don't have Docker installed, follow these steps.

**Install Docker:**
```bash
# Update package lists
sudo apt-get update

# Install prerequisites
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Engine, CLI, and Containerd
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Add your user to the `docker` group to run commands without `sudo`:**
```bash
sudo usermod -aG docker ${USER}
# You will need to log out and log back in for this to take effect.
newgrp docker
```

**Docker Compose is installed as a plugin with the command above.**

### 2. (Optional) Install Cloudflared for Public Access

If you want to expose your services to the internet securely, you can use Cloudflare Tunnels.

```bash
# Add cloudflare gpg key
sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

# Add cloudflare repository
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared jammy main' | sudo tee /etc/apt/sources.list.d/cloudflared.list

# install cloudflared
sudo apt-get update && sudo apt-get install cloudflared
```

## Getting Started

### 1. Clone the Repository

It is recommended to clone this repository into a directory like `/home/your-username/apps/`.

```bash
# Replace 'your-username' with your actual username
mkdir -p /home/your-username/apps
cd /home/your-username/apps

# Clone this repository
git clone <repository_url> openwebui-litellm
cd openwebui-litellm
```

### 2. Create the Environment File

This stack is configured using an `.env` file. Create a file named `.env` in the root of the project directory and populate it with the required values. A `.env.example` is provided; you can copy it to start.

**Create the `.env` file:**
```bash
cp .env.example .env
```
Now edit the `.env` file with your specific configuration.

**`.env` example:**
```dotenv
# .env

# --- PostgreSQL Database ---
# Set a strong, unique password for the LiteLLM database.
POSTGRES_PASSWORD=your_strong_postgres_password

# --- LiteLLM Configuration ---
# A master key to secure the LiteLLM API.
# Generate with: openssl rand -hex 32
LITELLM_MASTER_KEY=your_litellm_master_key

# A salt key used for hashing.
# Generate with: openssl rand -hex 32
LITELLM_SALT_KEY=your_litellm_salt_key

# --- LiteLLM UI Credentials (Optional) ---
# Set a username and password to protect the LiteLLM UI (cost tracking, etc.).
UI_USERNAME=admin
UI_PASSWORD=your_strong_ui_password

# --- OpenWebUI Configuration ---
# A secret key for securing OpenWebUI sessions.
# Generate with: openssl rand -hex 32
WEBUI_SECRET_KEY=your_webui_secret_key

# The name of your WebUI instance.
WEBUI_NAME=My AI Assistant

# The public URL of your OpenWebUI instance.
# Example: https://ai.yourdomain.com
WEBUI_URL=https://ai.yourdomain.com

# --- AI Provider API Keys ---
# Add API keys for the models you want to use.
# You only need to provide keys for the services you intend to use.
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
GEMINI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
XAI_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# --- S3 Storage (Optional) ---
# Configure S3-compatible storage for files and documents.
# Remove or leave these blank if not using S3.
STORAGE_PROVIDER=s3 # 'local', 's3', or 'gcs'
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
S3_ENDPOINT_URL=
S3_REGION_NAME=
S3_BUCKET_NAME=

# --- OneDrive Integration (Optional) ---
# For integrating with Microsoft OneDrive/SharePoint.
# Remove or leave these blank if not needed.
ONEDRIVE_SHAREPOINT_URL=https://yourtenant.sharepoint.com/
ENABLE_ONEDRIVE_INTEGRATION=False
MICROSOFT_CLIENT_ID=
MICROSOFT_CLIENT_SECRET=
MICROSOFT_CLIENT_TENANT_ID=
MICROSOFT_REDIRECT_URI=
```

You can generate strong random keys using `openssl rand -hex 32` on your terminal.

### 3. Start the Services

Once the `.env` file is configured, you can start all the services using Docker Compose.

```bash
docker compose up -d
```

This command will download the necessary Docker images and start the containers in the background (`-d` flag).

## Usage

### Accessing the Services

*   **OpenWebUI:** By default, OpenWebUI is accessible at `http://localhost:3080`. If you configured a public URL with Cloudflare, use that URL (e.g., `https://ai.yourdomain.com`).
*   **LiteLLM UI:** The LiteLLM UI for monitoring and cost tracking is accessible at `http://localhost:4000`. Use the `UI_USERNAME` and `UI_PASSWORD` from your `.env` file to log in.

### Managing the Stack

Here are some common Docker Compose commands:

*   **View container status:**
    ```bash
    docker compose ps
    ```
*   **View logs for all services:**
    ```bash
    docker compose logs -f
    ```
*   **View logs for a specific service (e.g., `litellm`):**
    ```bash
    docker compose logs -f litellm
    ```
*   **Stop and remove all containers, networks, and volumes:**
    ```bash
    docker compose down -v
    ```
    *Warning: This command will delete your LiteLLM and OpenWebUI data. Omit the `-v` flag to preserve data volumes.*
*   **Restart the services:**
    ```bash
    docker compose restart
    ```

## Configuration Details

### `docker-compose.yml`

This file defines the four services (`postgres`, `litellm`, `open-webui`, `watchtower`) and the network they communicate on. Key things to note:
*   **Ports:** LiteLLM is on `4000:4000` and OpenWebUI is on `3080:8080` (host:container).
*   **Volumes:** Data for each service is persisted in named Docker volumes (`postgres-data`, `litellm-data`, `open-webui`) to prevent data loss when containers are recreated.
*   **Healthchecks:** Ensure services are running correctly before dependent services start.

### `litellm_config.yaml`

This file contains configuration for LiteLLM. The provided configuration:
*   Sets the master key and UI credentials from environment variables.
*   Enables request streaming by default.
*   Forwards the `X-OpenWebUI-User-Name` header to track usage per user.
*   Allows cross-origin requests (`CORS`) from any domain (`"*"`). For better security, you should restrict this to your domain, e.g., `["https://litellm.yourdomain.com", "https://ai.yourdomain.com"]`.

### Exposing Services with Cloudflare Tunnels

Cloudflare Tunnels create a secure connection between your server and the Cloudflare network without opening any public inbound ports on your firewall.

**1. Authenticate `cloudflared`:**
```bash
cloudflared tunnel login
```
Follow the on-screen instructions to authorize the agent with your Cloudflare account.

**2. Create a Tunnel:**
Give your tunnel a name (e.g., `my-ai-stack`).
```bash
cloudflared tunnel create my-ai-stack
```
This command will output a Tunnel ID and the path to a credentials file. Note these down. The credentials file will be something like `/home/your-username/.cloudflared/<tunnel-id>.json`.

**3. Configure the Tunnel:**
Create a configuration file at `/home/your-username/.cloudflared/config.yml`.

**Example `config.yml`:**
```yaml
tunnel: <your-tunnel-id> # Paste your tunnel ID here
credentials-file: /home/your-username/.cloudflared/<your-tunnel-id>.json # Path to your credentials file

ingress:
  # Rule for OpenWebUI
  - hostname: ai.yourdomain.com
    service: http://localhost:3080
  
  # Rule for LiteLLM UI
  - hostname: litellm-ui.yourdomain.com  
    service: http://localhost:4000
    
  # Catch-all rule (sends traffic for other hostnames to a 404 page)
  - service: http_status:404
```

**Replace:**
*   `<your-tunnel-id>` with the ID from the `create` command.
*   `/home/your-username` with your home directory path.
*   `ai.yourdomain.com` and `litellm-ui.yourdomain.com` with your desired public hostnames.

**4. Point DNS Records to the Tunnel:**
In your Cloudflare DNS settings for `yourdomain.com`, create CNAME records for your hostnames pointing to your tunnel.
*   **Type:** `CNAME`
*   **Name:** `ai`
*   **Target:** `<your-tunnel-id>.cfargotunnel.com`
*   **Proxy status:** Proxied (Orange Cloud)

Do the same for `litellm-ui`.

**5. Run the Tunnel:**
You can run the tunnel as a service to ensure it's always running.
```bash
sudo cloudflared service install
sudo systemctl enable --now cloudflared
```
The tunnel will now start automatically on boot. Check its status with `systemctl status cloudflared`.

Now you can access OpenWebUI and LiteLLM via your public domain names.
