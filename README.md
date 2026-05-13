# adguard-cloud-homelab
A containerized, network-wide DNS sinkhole and privacy shield powered by AdGuard Home. Deployed on a Raspberry Pi 4 with secure remote management via Tailscale and dedicated persistent storage for audit logging and performance.
# Network-Wide DNS Sinkhole (AdGuard Home) 🛡️

**A high-performance network security and privacy layer for the modern homelab.**

This repository contains the deployment configuration for **AdGuard Home**, a network-wide DNS sinkhole that provides robust protection against tracking, advertisements, and malicious domains. By intercepting DNS requests at the network level, this solution secures all connected devices—from IoT hardware to mobile devices—without requiring individual client software.

### **Key Security Features:**
* **Privacy-First DNS:** Redirects traffic through a secure sinkhole to mitigate telemetry and data harvesting.
* **Modular Infrastructure:** Containerized via Docker to ensure isolation from other homelab services.
* **Zero-Exposure Networking:** Fully integrated with **Tailscale**, allowing for secure remote dashboard administration without exposing public ports to the internet.
* **Optimized Persistence:** Configured to utilize a dedicated 16GB ext4-formatted volume to protect the primary OS from log-heavy write operations.

---

## 🛠️ Architecture
* **Compute Node:** Raspberry Pi 4 Model B
* **Storage:** 16GB USB Flash Drive mounted at `/mnt/services` (ext4)
* **Networking:**  DNS queries handled via port `53` (TCP/UDP)
  * Setup Wizard accessible via Tailscale on port `3001`
  * Web Interface accessible via Tailscale on port `8081` (Bypassing default ports to prevent conflicts with adjacent services).

---

## 🚀 Deployment Instructions

### 1. Storage Preparation
Ensure the persistent storage drive is mounted and the necessary directories exist so Docker can properly map the volumes:

```bash
sudo mkdir -p /mnt/services/adguard/work
sudo mkdir -p /mnt/services/adguard/conf
sudo chown -R 1000:1000 /mnt/services/adguard
```

### 2. Stack Deployment
Navigate to the directory containing the `docker-compose.yml` stack and deploy the container in the background:

```bash
docker compose up -d
```

### 3. Initial Configuration
1. Open a web browser and access the initial setup wizard via the Tailscale network:
   `http://<TAILSCALE_IP>:3001`
2. **Critical Step:** During the wizard, on the "Listen Interfaces" screen, change the **Web Interface** port from `80` to **`8081`** to align with the Docker port mapping.
3. Once setup is complete, access the permanent dashboard at:
   `http://<TAILSCALE_IP>:8081`
