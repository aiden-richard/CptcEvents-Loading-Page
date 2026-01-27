# CptcEvents Loading Page

A containerized reverse proxy solution to mitigate cold start wait times for the [CPTC Events](https://github.com/aiden-richard/CptcEvents) web application.

## Overview

This repository deploys a reverse proxy stack that detects when the main CPTC Events application is starting up (cold start) and serves a polished loading page to users while the app initializes. The solution uses:

- **Cloudflare Tunnel** for secure, public internet access without exposing infrastructure
- **Caddy** as the reverse proxy, detects cold starts and serves a loading page
- **Interactive Loading Page** with animations, auto-refresh, and real-time status updates

### How It Works

```
User Browser (cptcevents.org)
        ↓
Cloudflare Tunnel
        ↓
Caddy Reverse Proxy (in Docker)
    ├─→ Azure Container App (Ready)
    │   └─→ Proxy request → User sees site
    │
    └─→ Azure Container App (Cold Start/Error)
        └─→ Serve loading page
        └─→ Client auto-refreshes every 3 seconds
        └─→ Once app ready → User automatically redirected
```

When a 502, 503, or 504 error is detected (or connection timeout), Caddy serves the loading page with a 200 status code. The loading page's JavaScript polls the server and automatically reloads once the app is healthy.

## Repository Structure

```
README.md                  # this file

Services/
├── .env                     # Cloudflare Tunnel token
├── docker-compose.yml       # Docker services configuration
└── proxy/
  ├── Caddyfile            # Caddy reverse proxy configuration
  └── coldstart/
    └── loading.html     # Interactive loading page with animations
```

## Quick Start

### Prerequisites

- Docker and Docker Compose installed
- Cloudflare account with a domain configured
- Cloudflare Tunnel created in Cloudflare One dashboard (Networks > Tunnels)
- Cloudflare Tunnel token (obtained from dashboard)
- Application route configured for the tunnel in Cloudflare
- Azure Container App running with a public URL
- Linux server or local machine to run Docker

### Setup Steps

1. **Clone & Configure**
   ```bash
   git clone <repo>
   cd Services
   ```

2. **Create .env and Set Token**
   ```
   TOKEN={your_cloudflare_tunnel_token}
   ```
   
3. **Configure Cloudflare DNS**
   - Log in to Cloudflare dashboard
   - Select your domain
   - Go to DNS settings
   - Ensure your domain nameservers are pointing to Cloudflare
   - The tunnel will automatically create the required DNS records (CNAME records)
   - Verify the tunnel appears in **Access > Tunnels** section of Cloudflare dashboard

4. **Configuration Files**
   - `proxy/Caddyfile` to point to the Azure Container App URL

5. **Start Services**
   ```bash
   # Create Docker network (if not created yet)
   docker network create CptcEventsNetwork

   # Start containers
   docker compose up -d
   ```

6. **Verify**
   - Check container logs: `docker logs {container_name} -f`
   - Visit the site URL to see the loading page or live site

## Technologies

- **Container Platform**: Docker & Docker Compose
- **Reverse Proxy**: Caddy (lightweight, auto-HTTPS capable)
- **Tunneling**: Cloudflare Tunnel (zero-trust access)
- **Loading Page**: HTML5, vanilla JavaScript, CSS animations
- **Upstream**: Azure Container Apps (ASP.NET Core backend)

## Related Projects

- **[CPTC Events](https://github.com/aiden-richard/CptcEvents)** - Main application
  - ASP.NET Core 10.0 backend
  - FullCalendar event management
  - Azure Container Apps hosting

## See Also

- [Cloudflare Tunnel Docs](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/)

## License

Same as [CPTC Events](https://github.com/aiden-richard/CptcEvents)
