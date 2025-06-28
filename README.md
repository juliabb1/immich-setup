# Setting Up Immich with Port Forwarding, Dynamic DNS, and SSL Encryption

This guide will help you set up **Immich** on your Raspberry Pi as a self-hosted photo and video cloud solution, accessible securely over the internet with HTTPS encryption. It covers the complete setup including port forwarding, dynamic DNS configuration, and SSL certificates managed automatically via Caddy reverse proxy.

---

## Overview: HTTP vs HTTPS

| Protocol | Port | Encryption | Description |
|----------|------|------------|-------------|
| **HTTP** | 80   | ❌ None     | Sends data in plain text, vulnerable to interception. |
| **HTTPS** | 443 | ✅ SSL/TLS | Encrypts data in transit and verifies website identity. |

---

## What is Caddy and a Reverse Proxy?
**Caddy** is a lightweight web server that:
- Automatically manages SSL/TLS certificates using Let’s Encrypt.
- Functions as a **reverse proxy** to route traffic to internal services.
- Listens on ports **80** (HTTP) and **443** (HTTPS).
- Redirects all HTTP traffic to HTTPS by default.

**Reverse Proxy Flow:**
1. Client requests `https://your-domain.com`.
2. Caddy receives the request and handles SSL.
3. Caddy forwards the request to your local Immich container (`http://immich-server:port`).
4. The response is returned to Caddy.
5. Caddy encrypts and sends it securely to the user over HTTPS.

---

## Setup Instructions

### Prerequisites
- Raspberry Pi (64-bit OS)
- Docker installed
- microSD card for storage
- Access to router for port forwarding
- Dynamic DNS account (e.g. [no-ip.com](https://no-ip.com))

---

### Step 1: Configure Port Forwarding & Docker Network

1. Log in to your router (e.g. `fritz.box`)  
2. **Forward ports 80 (HTTP) and 443 (HTTPS)** to your Raspberry Pi's local IP.  
3. Set up a **DynDNS** hostname for your public IP.  
4. On your Pi, **create a shared Docker network**:  
   `docker network create shared_net`

### Step 2: Set Up Caddy Reverse Proxy
1. Create a `docker-compose.yml` for Caddy following [Caddy's docker compose example](https://caddyserver.com/docs/running#docker-compose)
Add your custom Docker network in the compose file:
```
services:
  caddy:
    ...
    networks:
      - caddy_net

networks:
  caddy_net:
    external: true
    name: {network_name}
```
2. Run the Caddy Container `docker compose up -d`
3. In the same directory as the compose file create the configuration directory `conf` and Caddyfile `Caddyfile` with the following content
```
{your_domain.net} {
    reverse_proxy {immich_server_container_name}:{immich_server_container_port}
}
// Replace placeholders with your actual domain, Immich container name, and port.
```
4. Restart or start Caddy to apply configuration with `docker compose up -d`


### Step 3: Set Up Immich on Docker
1. Follow the instructions on [Immich's quick start guide](https://immich.app/docs/overview/quick-start/).
2. Configure your `.env` file and set `UPLOAD_LOCATION` to the desired storage path on your microSD card.
3. Modify `Immich docker-compose.yml` to join the docker network by adding the following under each relevant service:
```
services:
  immich-server:
    container_name: immich_server
    ...
    networks:
      - caddy_net

...

networks:
  caddy_net:
    external: true
    name: {network_name}
```
4. Run Immich Containers `docker compose up -d`


### Step 4: Enjoy your HTTPS-encrypted Immich Cloud
Opern your Browser and navigate to `https://{your_domain.net}`
![image](https://github.com/user-attachments/assets/469388f3-69b7-465c-9b91-89b219e6149f)

# Summary on the HTTPS setup via Caddy
This overview illustrates how Caddy enables HTTPS encryption for your Dockerized services by acting as a reverse proxy:
Router forwards ports 80 and 443 to your Raspberry Pi.
Caddy listens on these ports, automatically obtaining and renewing SSL certificates.
Caddy securely forwards requests to Immich backend containers on your Docker network.

![image](https://github.com/user-attachments/assets/3d5a2ad1-ddea-44ec-a8ef-262cb504d7eb)

# Helpful Commands

- `docker ps`  
  List running Docker containers.
- `docker compose up -d`  
  Start containers in detached mode.
- `docker compose down`  
  Stop and remove containers, networks, images, and volumes.
- `docker network create net_name`  
  Create a Docker network with the specified name.
- `docker logs container_name`  
  View logs of a specific container.

