# Docker Services

The Ubuntu Server VM acts as the primary Docker host for the homelab application environment.

Docker services are deployed using Docker Compose, with each application maintained in its own directory.

## Architecture

```text
Ubuntu Server
10.0.0.60
│
└── Docker Engine
    │
    ├── Caddy
    ├── Jellyfin
    ├── Navidrome
    ├── Nextcloud
    ├── Kavita
    ├── Portainer
    ├── Uptime Kuma
    └── Homelab Dashboard
```

## Services

| Directory      | Service                 | Purpose                 |
| -------------- | ----------------------- | ----------------------- |
| `caddy/`       | Caddy                   | HTTPS and reverse proxy |
| `jellyfin/`    | Jellyfin                | Movie/media streaming   |
| `navidrome/`   | Navidrome               | Music streaming         |
| `nextcloud/`   | Nextcloud               | Private file storage    |
| `kavita/`      | Kavita                  | Ebook/PDF library       |
| `portainer/`   | Portainer CE            | Docker management       |
| `uptime-kuma/` | Uptime Kuma             | Service monitoring      |
| `dashboard/`   | Nginx + custom HTML/CSS | Homelab dashboard       |

## Deployment Model

Each service uses Docker Compose to define its container configuration.

Configurations demonstrate practical use of:

* Docker images
* Containers
* Port mappings
* Persistent volumes
* Bind mounts
* Environment variables
* Restart policies
* Container networking

The configurations are kept separate by service to make deployments easier to manage and troubleshoot.

## Reverse Proxy

Caddy provides the primary application-facing HTTPS layer.

For example:

```text
https://jellyfin.homelab
        │
        ▼
      Caddy
        │
        ▼
10.0.0.60:8096
```

Similar routing is configured for the other homelab applications.

## Persistent Data

Application configuration and important service data are stored using Docker volumes or bind mounts rather than relying on the temporary container filesystem.

Media is maintained separately from application configuration.

The repository contains deployment configuration only. Actual media, databases, personal files, and application runtime data are intentionally not included.

## Security

Sensitive values should not be committed to the repository.

Environment-specific credentials, API keys, private keys, and other secrets should be supplied locally through environment variables or other appropriate configuration mechanisms.

The Compose files in this repository are intended to document and reproduce the infrastructure configuration rather than expose private runtime data.
