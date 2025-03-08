# Self-Hosted RustDesk Server on Oracle Cloud Free Tier

This documentation details how to set up a self-hosted RustDesk server on an Oracle Cloud Infrastructure (OCI) Free Tier VM using Docker and Nginx as a reverse proxy with SSL. Follow this guide to deploy RustDesk securely and enable remote desktop connections without exposing sensitive details.

> **Note:**  
> All references to IP addresses, domain names, and keys are generic. Replace placeholders with your own values during your setup. This guide assumes you are using an Oracle Free Tier instance (e.g., a `VM.Standard.E2.1.Micro` shape) and a domain of your choosing.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [OCI VM Setup on Free Tier](#oci-vm-setup-on-free-tier)
- [Firewall and Ingress Configuration](#firewall-and-ingress-configuration)
  - [OCI Security List Ingress Rules](#oci-security-list-ingress-rules)
- [Docker Compose Setup for RustDesk](#docker-compose-setup-for-rustdesk)
- [Reverse Proxy & DNS Setup](#reverse-proxy--dns-setup)
- [RustDesk Server & Client Configuration](#rustdesk-server--client-configuration)
- [Additional Notes](#additional-notes)

---

## Overview

RustDesk is an open-source remote desktop solution that uses two main services:

- **hbbs (ID/Rendezvous Server):** Allows clients to discover and negotiate connections.
- **hbbr (Relay Server):** Relays traffic when a direct connection is not possible.

This guide will help you deploy these services on an OCI Free Tier VM, secure them with SSL using Nginx, and configure your clients.

---

## Prerequisites

- An Oracle Cloud Infrastructure account (OCI Free Tier eligible).
- A Linux-based VM instance on OCI (e.g., Ubuntu Server 22.04 LTS or Oracle Linux).
- A domain name for your reverse proxy (e.g., `<YOUR_DOMAIN>`).
- Basic knowledge of SSH, Docker, and network/firewall management.
- Docker and Docker Compose installed on your VM.

---

## OCI VM Setup on Free Tier

1. **Launch Your OCI Instance:**
   - Log in to your [OCI Console](https://cloud.oracle.com/).
   - Navigate to **Compute > Instances** and create a new instance.
   - Choose a Linux image (e.g., Ubuntu Server 22.04 LTS) and select the **VM.Standard.E2.1.Micro** shape (Always Free eligible).

2. **Access Your VM:**
   - Connect via SSH:
     ```bash
     ssh -i /path/to/your/private_key username@<YOUR_VM_PUBLIC_IP>
     ```
   - Replace `/path/to/your/private_key` and `<YOUR_VM_PUBLIC_IP>` with your key file and assigned public IP, respectively.

3. **Install Docker and Docker Compose:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install docker.io docker-compose -y
   ```

---

## Firewall and Ingress Configuration

### OCI Security List Ingress Rules

In OCI, configure your Virtual Cloud Network (VCN) with Security List rules to allow the necessary traffic:

- **Rule 1: Allow TCP Traffic for RustDesk**
  - **Source:** `0.0.0.0/0`
  - **Protocol:** TCP
  - **Destination Port Range:** `21115-21119`
  
- **Rule 2: Allow UDP Traffic for RustDesk**
  - **Source:** `0.0.0.0/0`
  - **Protocol:** UDP
  - **Destination Port Range:** `21116`

- **Rule 3: Allow HTTP Traffic**
  - **Source:** `0.0.0.0/0`
  - **Protocol:** TCP
  - **Destination Port:** `80`

- **Rule 4: Allow HTTPS Traffic**
  - **Source:** `0.0.0.0/0`
  - **Protocol:** TCP
  - **Destination Port:** `443`

*These rules ensure that the RustDesk services and the Nginx reverse proxy are accessible from the internet.*

### Local Firewall Configuration (Using UFW on Ubuntu)

On your VM, allow the same ports:
```bash
sudo ufw allow 21115/tcp
sudo ufw allow 21116/tcp
sudo ufw allow 21116/udp
sudo ufw allow 21117/tcp
sudo ufw allow 21118/tcp
sudo ufw allow 21119/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

---

## Docker Compose Setup for RustDesk

1. **Create a Working Directory:**
   ```bash
   sudo mkdir -p /opt/rustdesk
   cd /opt/rustdesk
   ```

2. **Create `docker-compose.yml`:**
   ```yaml
   version: '3'
   services:
     hbbs:
       container_name: hbbs
       image: rustdesk/rustdesk-server:latest
       command: hbbs -r 127.0.0.1:21117 -k _
       ports:
         - "21115:21115"
         - "21116:21116"
         - "21116:21116/udp"
         - "21118:21118"
       volumes:
         - ./hbbs:/root
       depends_on:
         - hbbr
       restart: unless-stopped

     hbbr:
       container_name: hbbr
       image: rustdesk/rustdesk-server:latest
       command: hbbr -k _
       ports:
         - "21117:21117"
         - "21119:21119"
       volumes:
         - ./hbbr:/root
       restart: unless-stopped
   ```

3. **Deploy the Containers:**
   ```bash
   sudo docker-compose up -d
   ```
4. **Verify Deployment:**
   ```bash
   sudo docker ps
   ```

---

## Reverse Proxy & DNS Setup

### 1. DNS Configuration

- **Log in to your DNS provider.**
- **Create an A Record:**
  - **Name:** Use a subdomain (e.g., `support`)
  - **Type:** A
  - **Value:** Your VM’s public IP (replace with `<YOUR_VM_PUBLIC_IP>`)
- **Allow Time for Propagation:**  
  Verify that your subdomain resolves correctly using tools like [WhatsMyDNS](https://www.whatsmydns.net/).

### 2. Nginx Reverse Proxy Setup

1. **Install Nginx:**
   ```bash
   sudo apt update
   sudo apt install nginx -y
   ```
2. **Create a Configuration File:**
   ```bash
   sudo nano /etc/nginx/conf.d/rustdesk.conf
   ```
3. **Add the Following Configuration:**
   ```nginx
   server {
       listen 80;
       server_name <YOUR_DOMAIN>;

       # Redirect all HTTP traffic to HTTPS
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl;
       server_name <YOUR_DOMAIN>;

       # SSL configuration (certificates obtained via Certbot)
       ssl_certificate /etc/letsencrypt/live/<YOUR_DOMAIN>/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/<YOUR_DOMAIN>/privkey.pem;
       include /etc/letsencrypt/options-ssl-nginx.conf;
       ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

       # Main reverse proxy for RustDesk web interface
       location / {
           proxy_pass http://127.0.0.1:21114/;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }

       # Reverse proxy for WebSocket endpoints
       location /ws/id {
           proxy_pass http://127.0.0.1:21118;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "Upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }

       location /ws/relay {
           proxy_pass http://127.0.0.1:21119;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "Upgrade";
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
   }
   ```
   Replace `<YOUR_DOMAIN>` with your chosen domain (e.g., `support.yourdomain.com`).

4. **Test and Reload Nginx:**
   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. **Obtain an SSL Certificate with Certbot:**
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   sudo certbot --nginx -d <YOUR_DOMAIN>
   ```
   Follow the prompts to complete certificate issuance.

---

## RustDesk Server & Client Configuration

### Server Configuration

1. **Retrieve the Public Key:**
   ```bash
   sudo cat /opt/rustdesk/hbbs/id_ed25519.pub
   ```
   Copy the output (this is needed for client configuration).

2. **Ensure Services Are Running:**
   ```bash
   sudo docker ps
   ```

### Client Configuration

1. **Download and Install the RustDesk Client:**
   - For desktop or mobile, visit the [RustDesk Downloads](https://rustdesk.com/download) page.

2. **Configure Network Settings:**
   - Open **Settings** in the client and go to the **Network** tab.
   - **ID Server:** Set to `<YOUR_DOMAIN>` (e.g., `support.yourdomain.com`).
   - **Key:** Paste the public key from your server.
   - Leave the Relay Server and API fields blank unless using a custom setup.
   - Save your settings.

3. **Test the Connection:**
   - Note the unique RustDesk ID on the client.
   - Use another client to connect by entering the target device’s ID.

---

## Additional Notes

- **Certificate Renewal:**  
  Certbot auto-renews certificates. Test with:
  ```bash
  sudo certbot renew --dry-run
  ```
- **Monitoring:**  
  Monitor Docker logs for RustDesk services:
  ```bash
  sudo docker logs hbbs
  sudo docker logs hbbr
  ```
