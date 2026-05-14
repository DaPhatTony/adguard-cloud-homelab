# AdGuard Cloud Homelab: Secure DNS Sinkhole & Privacy Shield

This repository contains the deployment configuration for a containerized, network-wide DNS sinkhole powered by AdGuard Home. Deployed on a Raspberry Pi 4, this project emphasizes secure remote management and encrypted DNS protocols (DoH/DoT) over a zero-trust network.

## Architecture & Security Overview
This infrastructure integrates robust network privacy and security practices:
* **Containerization:** The AdGuard Home service and its configurations are completely isolated using Docker.
* **Encrypted DNS:** Configured to support DNS-over-HTTPS (DoH), DNS-over-TLS (DoT), and DNS-over-QUIC (DoQ) to prevent upstream DNS hijacking and ISP snooping.
* **Dashboard Security:** The administrative interface is secured with full SSL/TLS termination using Tailscale MagicDNS certificates.
* **Custom Port Binding:** HTTPS traffic is routed through port `8443` to ensure clean coexistence with other secure services (like Vaultwarden and Nginx) running on the same host.
* **Network Isolation:** Administrative access is restricted strictly to authenticated Tailscale VPN clients.

## System Prerequisites
* **Hardware:** Raspberry Pi 4
* **OS:** Raspberry Pi OS (64-bit)
* **Dependencies:** Docker Engine, Docker Compose V2
* **Network:** Tailscale installed and authenticated with MagicDNS enabled

---

## Step-by-Step Installation Guide

### Step 1: Initialize the Environment
Create an isolated directory and set up version control to track infrastructure changes.
```bash
mkdir ~/adguard-cloud-homelab
cd ~/adguard-cloud-homelab
git init
```

### Step 2: Establish Security Exclusions
Prevent private SSL keys and personal network configurations from being pushed to public repositories.
```bash
nano .gitignore
```
Add the following rules:
```text
certs/
workdir/
confdir/
```

### Step 3: SSL/TLS Certificate Generation
Generate domain-specific certificates using Tailscale to encrypt the administrative dashboard and DNS queries.
```bash
mkdir certs
sudo tailscale cert --cert-file ./certs/tls.crt --key-file ./certs/tls.key <your-tailscale-domain>
```

### Step 4: Configure the Docker Infrastructure
Define the stack, mapping the necessary DNS ports, the updated HTTPS port (`8443`), and the certificate volumes.
```bash
nano docker-compose.yml
```

```yaml
services:
  adguardhome:
    image: adguard/adguardhome
    container_name: adguardhome
    restart: unless-stopped
    volumes:
      - ./workdir:/opt/adguardhome/work
      - ./confdir:/opt/adguardhome/conf
      - ./certs:/certs:ro
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8081:8081/tcp"
      - "3001:3000/tcp"
      - "8443:443/tcp"
      - "8443:443/udp"
```

### Step 5: Execute Deployment
Launch the secure stack in detached mode.
```bash
docker compose up -d
```

### Step 6: Internal Encryption Setup
1. Access the web interface via the initial HTTP setup port (`3001`).
2. Navigate to **Settings** > **Encryption Settings**.
3. Enable encryption and map the `/certs/tls.crt` and `/certs/tls.key` paths.

---

## Secure Access
Once encryption is validated, the dashboard can be managed securely across the Tailscale network:
* **HTTPS Access:** `https://<tailscale-machine-name>.<tailnet-id>.ts.net:8443`
