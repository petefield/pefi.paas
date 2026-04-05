# pefi.paas

A Docker Compose configuration for quickly standing up and exposing services made up of Docker images.

## Services

| Service | Image | Description |
|---|---|---|
| **MongoDB** | `mongo:8` | Database used by Pefi.ServiceManager for persistence |
| **RabbitMQ** | `rabbitmq:3-management` | Message broker used for inter-service communication |
| **Pefi.ServiceManager** | `ghcr.io/petefield/pefi.servicemanager:latest` | Manages Docker containers/services via REST API |
| **Pefi.Proxy** | `ghcr.io/petefield/pefi.proxy:latest` | Reverse proxy that routes traffic to registered services |
| **Pefi.DynamicDNS** | `ghcr.io/petefield/pefi.dynamicdns:latest` | Keeps a DNSimple DNS record updated with your current public IP |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) with the Compose plugin
- A [DNSimple](https://dnsimple.com) account and API token (required for Pefi.DynamicDNS)
- Access to the GitHub Container Registry packages (the images are published to `ghcr.io/petefield/`)

## Getting started

1. **Clone the repository**

   ```bash
   git clone https://github.com/petefield/pefi.paas.git
   cd pefi.paas
   ```

2. **Create your environment file**

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and fill in your values – at a minimum you must set the `DNS_*` variables for Pefi.DynamicDNS:

   | Variable | Description | Default |
   |---|---|---|
   | `RABBITMQ_USER` | RabbitMQ username | `guest` |
   | `RABBITMQ_PASS` | RabbitMQ password | `guest` |
   | `DNS_DOMAIN` | Your DNSimple domain (e.g. `example.com`) | *(required)* |
   | `DNS_HOME_HOSTNAME` | The hostname to keep updated (e.g. `home.example.com`) | *(required)* |
   | `DNS_API_TOKEN` | Your DNSimple API token | *(required)* |
   | `DNS_CHECK_INTERVAL_MINUTES` | How often to check the public IP | `2` |

3. **Authenticate with the GitHub Container Registry**

   ```bash
   echo YOUR_GITHUB_TOKEN | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
   ```

4. **Start the stack**

   ```bash
   docker compose up -d
   ```

## Exposed ports

| Port | Service |
|---|---|
| `80` | Pefi.Proxy (reverse proxy – entry point for your services) |
| `9090` | Pefi.ServiceManager REST API |
| `15672` | RabbitMQ Management UI |

## Architecture

```
Internet
   │
   ▼
Pefi.Proxy (:80)
   │  (routes traffic to registered services)
   ▼
Managed Docker containers
   ▲
   │  (creates/stops/restarts containers)
Pefi.ServiceManager (:9090)
   │
   ├── MongoDB  (service registry / persistence)
   └── RabbitMQ (events: service created, IP changed, …)
         ├── Pefi.Proxy        (listens for service-created events)
         └── Pefi.DynamicDNS   (listens for IP-change events)
```
