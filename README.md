# AI Inference Stack (vLLM + LiteLLM + Open WebUI)

This project provides a production-ready deployment for a self-hosted Large Language Model (LLM) ecosystem. It combines a high-performance inference engine, a management gateway, and an enterprise-grade user interface.

## 🏗 Architecture Overview

The system consists of four interconnected services:

1. **vLLM (Inference Engine):** Runs the Gemma-4 model. It is optimized for high throughput using FP8 quantization and efficient KV caching.
2. **PostgreSQL (Database):** Provides persistent storage for LiteLLM, handling API key management, usage tracking, and configuration.
3. **LiteLLM (Proxy Gateway):** Acts as an OpenAI-compatible middleware. It handles load balancing, detailed logging, and abstracts the backend engine from the frontend.
4. **Open WebUI (Frontend):** A professional chat interface that supports multi-user management and integrates with LDAP/Active Directory for enterprise authentication.

---

## 🚀 Deployment Guide

### Prerequisites

- **Hardware:** NVIDIA GPU with sufficient VRAM to support the Gemma-4 31B model.
- **Drivers:** Latest NVIDIA Drivers and the NVIDIA Container Toolkit installed.
- **Software:** 
  - Docker Engine
  - Docker Compose

### Installation Steps

1. **Clone the repository** or create your project directory:
   `mkdir ai-inference-stack && cd ai-inference-stack`

2. **Configure Environment Variables:**
   Create a `.env` file in the root directory using this template:

   # vLLM
   HF_TOKEN=your_huggingface_token_here

   # Database
   POSTGRES_PASSWORD=secure_db_password

   # LiteLLM
   LITELLM_MASTER_KEY=sk-your-master-key
   LITELLM_UI_USER=admin
   LITELLM_UI_PASSWORD=secure_admin_password
   LITELLM_WEBUI_KEY=sk-webui-api-key

   # Open WebUI
   WEBUI_SECRET_KEY=generate_a_long_random_string
   SERVER_HOST=localhost

   # LDAP / Active Directory
   LDAP_HOST=ldap.example.com
   LDAP_PORT=389
   LDAP_APP_DN=cn=admin,dc=example,dc=com
   LDAP_APP_PASSWORD=ldap_password
   LDAP_SEARCH_BASE=dc=example,dc=com
   LDAP_SEARCH_FILTER=(sAMAccountName=*)

3. **Configuration File:**
   Ensure your `litellm-config.yaml` is placed at `/srv/app/litellm-config.yaml` on the host machine. This file defines the routing between the proxy and the vLLM engine.

4. **Launch the Stack:**
   `docker compose up -d`

---

## ⚙️ Technical Specifications

### Network Isolation
To ensure security, the stack is split into two Docker networks:
- **ai_internal**: A restricted network for the Database and vLLM engine. These services cannot be reached from outside the Docker host.
- **ai_external**: A bridge network that exposes the LiteLLM Proxy (Port 4000) and Open WebUI (Port 3000) to the local network.

### Service Matrix

| Service | Port | External Access | Role |
| :--- | :--- | :--- | :--- |
| vLLM | 8003 | No | Model Execution |
| PostgreSQL | 5432 | No | Data Persistence |
| LiteLLM | 4000 | Yes | API Gateway |
| Open WebUI | 3000 | Yes | User Interface |

---

## 🔐 Authentication & Access

### LDAP / Active Directory
The system is configured for enterprise access:
- **Sign-ups:** Disabled (`ENABLE_SIGNUP=False`) to prevent unauthorized account creation.
- **Identity:** Uses `sAMAccountName` for usernames and `mail` for email attributes.
- **Permissions:** Group management is enabled via the `memberOf` attribute, allowing for role-based access control based on AD groups.

---

## 🛠 Management & Maintenance

### Monitoring Logs
`docker compose logs -f` (All services)
`docker compose logs -f gemma-4` (Inference engine only)

### Performance Tuning
If the system encounters Out-Of-Memory (OOM) issues, adjust the following values in `docker-compose.yaml`:
- `gpu-memory-utilization`: Lower from `0.92` to `0.85` or `0.90`.
- `max-model-len`: Reduce the context window (e.g., from `201000` to `32768`) to save VRAM.

### Updating the Stack
To apply changes to the `.env` file or the Compose configuration:
`docker compose up -d --force-recreate`
