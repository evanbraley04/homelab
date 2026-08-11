# Personal IT Homelab

A self-hosted infrastructure environment built on a Dell OptiPlex 5080 Micro using Proxmox VE. The project combines Linux administration, virtualization, Docker, networking, DNS, VPNs, reverse proxies, TLS/PKI, service monitoring, and a Windows Server Active Directory environment.

The goal of the project is to gain practical experience building, configuring, securing, and troubleshooting the types of infrastructure commonly encountered in IT and systems administration environments.

---

## Architecture

The homelab is built around a Dell OptiPlex 5080 Micro running Proxmox VE as the bare-metal hypervisor.

The primary Ubuntu VM hosts the Docker-based applications, while dedicated LXC containers provide networking and infrastructure services. A separate Windows Server/Windows 11 environment provides an isolated Active Directory lab.

### Main Homelab

```text
Internet
   │
   ▼
Xfinity Router
10.0.0.1
   │
   │ 10.0.0.0/24
   │
   ├──────────────────────────────────────────────┐
   │                                              │
   ▼                                              ▼
Proxmox VE                                  Home Network Devices
Dell OptiPlex 5080
   │
   ├── Ubuntu Docker VM
   │   10.0.0.60
   │   │
   │   ├── Caddy
   │   │   └── HTTPS / Reverse Proxy
   │   │
   │   ├── Jellyfin
   │   ├── Navidrome
   │   ├── Nextcloud
   │   ├── Kavita
   │   ├── Portainer
   │   ├── Uptime Kuma
   │   └── Homelab Dashboard
   │
   ├── Tailscale LXC
   │   10.0.0.70
   │   └── Subnet Router
   │
   └── Pi-hole LXC
       10.0.0.80
       └── DNS / Ad Blocking
```

### Windows / Active Directory Lab

```text
Xfinity Router
10.0.0.1
      │
      ▼
   vmbr0
      │
      ├── DC01
      │   10.0.0.90
      │   Windows Server 2025
      │   │
      │   ├── Active Directory Domain Services
      │   ├── DNS
      │   └── Group Policy
      │
      └── CLIENT01
          10.0.0.100
          Windows 11
          │
          └── Joined to windowslab.local
```

Full architecture diagrams are available in [`diagrams/`](diagrams/).

---

## Infrastructure

| Component            | Technology               | Purpose                           |
| -------------------- | ------------------------ | --------------------------------- |
| Physical Server      | Dell OptiPlex 5080 Micro | Homelab hardware                  |
| Hypervisor           | Proxmox VE               | Virtualization and container host |
| Docker Host          | Ubuntu Server 26.04 LTS  | Hosts Docker applications         |
| VPN                  | Tailscale                | Remote access and subnet routing  |
| DNS                  | Pi-hole                  | Local DNS and ad blocking         |
| Reverse Proxy        | Caddy                    | HTTPS and internal reverse proxy  |
| Monitoring           | Uptime Kuma              | Service availability monitoring   |
| Container Management | Portainer CE             | Docker administration             |
| Windows Server       | Windows Server 2025      | Active Directory infrastructure   |
| Windows Client       | Windows 11               | Domain workstation                |
| Web Server           | Nginx                    | Hosts custom dashboard            |
| Database             | MariaDB                  | Nextcloud database                |

---

## Docker Services

The Ubuntu Docker VM hosts the primary application stack using Docker Compose.

### Jellyfin

Self-hosted media streaming server.

* Movie library
* MKV media
* Automatic metadata and poster retrieval
* Genres and library organization
* Continue Watching
* Playback testing

### Navidrome

Self-hosted music streaming server.

* FLAC music library
* Artist/album organization
* Embedded metadata
* Album artwork
* Synchronized `.lrc` lyrics
* Subsonic-compatible ecosystem

### Nextcloud

Self-hosted file and personal data platform.

* Private file storage
* Personal journaling
* MariaDB backend
* Persistent Docker volumes
* Local-only infrastructure

### Kavita

Self-hosted ebook and document library.

* PDF library
* Individual book organization
* Metadata/library scanning
* Remote access through Tailscale

### Portainer

Web-based Docker management.

Used to manage:

* Containers
* Images
* Volumes
* Networks
* Compose stacks
* Container lifecycle

### Uptime Kuma

Service monitoring platform used to monitor the availability of homelab services.

Monitors include:

* HTTP/HTTPS services
* Pi-hole
* Internal Caddy endpoints
* Other infrastructure services

### Homelab Dashboard

A custom static website built using HTML and CSS.

The dashboard provides a central landing page for:

* Movies
* Music
* Books
* Journal
* DNS / Ad Blocking
* VPN
* Docker Management
* Monitoring

It is deployed using Nginx and exposed internally through Caddy.

---

## Networking

The homelab uses the existing `10.0.0.0/24` home LAN.

Important infrastructure addresses include:

| Address      | Device           | Purpose                |
| ------------ | ---------------- | ---------------------- |
| `10.0.0.1`   | Xfinity Router   | Gateway                |
| `10.0.0.50`  | Proxmox          | Hypervisor             |
| `10.0.0.60`  | Ubuntu Docker VM | Docker services        |
| `10.0.0.70`  | Tailscale LXC    | VPN subnet router      |
| `10.0.0.80`  | Pi-hole LXC      | DNS                    |
| `10.0.0.90`  | DC01             | Active Directory / DNS |
| `10.0.0.100` | CLIENT01         | Windows workstation    |

The homelab is intentionally **not publicly exposed to the Internet**.

Remote access is provided through Tailscale.

---

## Remote Access

Tailscale is configured as a subnet router on a dedicated Debian LXC.

The LXC advertises:

```text
10.0.0.0/24
```

This allows remote Tailscale devices to access services on the home LAN without installing Tailscale directly into every Docker container.

For example:

```text
Remote MacBook
      │
      │ Encrypted Tailscale tunnel
      ▼
Tailscale LXC
10.0.0.70
      │
      │ LAN routing
      ▼
Ubuntu Docker VM
10.0.0.60
      │
      ▼
Docker service
```

Remote access was successfully tested from a MacBook connected through a phone hotspot.

No router port forwarding is required.

---

## DNS

Pi-hole provides DNS filtering and local DNS records for the homelab.

Internal service names resolve to the Ubuntu Docker VM:

```text
jellyfin.homelab    → 10.0.0.60
navidrome.homelab   → 10.0.0.60
nextcloud.homelab   → 10.0.0.60
kavita.homelab      → 10.0.0.60
portainer.homelab   → 10.0.0.60
dashboard.homelab   → 10.0.0.60
```

Pi-hole also provides ad/domain blocking using its gravity database.

Tailscale DNS is configured so remote Tailscale devices can use Pi-hole without changing the DNS configuration of the household router.

---

## HTTPS and Reverse Proxy

Caddy provides the HTTPS endpoint for the homelab services.

The normal application path is:

```text
Client
  │
  │ HTTPS
  ▼
Caddy
  │
  │ HTTP
  ▼
Docker Application
```

Applications are accessed using friendly URLs rather than manually entering Docker ports:

```text
https://jellyfin.homelab
https://navidrome.homelab
https://nextcloud.homelab
https://kavita.homelab
https://portainer.homelab
https://dashboard.homelab
```

Caddy uses an internal certificate authority for the `.homelab` certificates.

Trusted client devices have the Caddy root CA installed.

Portainer is a special case because its backend endpoint already uses HTTPS, so Caddy connects to it over HTTPS while explicitly handling its internal certificate verification.

---

## Windows / Active Directory Lab

A separate Windows environment was created to practice enterprise Windows administration.

### Domain

```text
windowslab.local
```

### Domain Controller

```text
DC01
10.0.0.90
Windows Server 2025
```

Roles:

* Active Directory Domain Services
* DNS Server
* Group Policy Management
* Global Catalog

### Workstation

```text
CLIENT01
10.0.0.100
Windows 11
```

CLIENT01 is joined to the `windowslab.local` domain and uses DC01 as its DNS server.

### Active Directory Structure

```text
windowslab.local
│
├── Lab-Users
├── Lab-Groups
├── Lab-Computers
├── Lab-Servers
└── Lab-Workstations
```

The lab includes domain users and security groups used to practice identity management and permissions.

---

## Group Policy

Several Group Policy Objects were created and tested.

### Lab-Workstation-Policy

Applied to `Lab-Workstations`.

Configured to disable the Windows Shutdown Event Tracker.

### Lab-Workstation-IT-Admins

Applied to workstation computers.

Uses Group Policy Preferences to add the domain security group:

```text
WINDOWSLAB\Lab-IT-Admins
```

to the local Administrators group on domain workstations.

This demonstrates how workstation administrative access can be delegated through a domain security group without making individual users Domain Administrators.

### Lab-User-Policy

Applied to users in the `Lab-Users` security group.

Configured to prevent access to:

* Control Panel
* Windows PC Settings

---

## Troubleshooting Experience

A major purpose of the project is to troubleshoot failures rather than simply build configurations that work on the first attempt.

### Group Policy Filtering

The user-targeted GPO initially failed to apply.

`gpresult` reported:

```text
Lab-User-Policy
Filtering: Not Applied
```

The following were investigated:

1. User OU placement
2. GPO link
3. GPO link status
4. GPO enabled status
5. Security filtering
6. Read permissions
7. Apply Group Policy permissions
8. Group membership
9. Domain authentication
10. Policy refresh

The issue was eventually traced to the computer account lacking permission to read the GPO.

The final permissions were:

```text
Lab-Users
├── Read
└── Apply Group Policy

Domain Computers
└── Read
```

After correcting the permissions and running:

```text
gpupdate /force
```

the policy successfully applied.

This provided practical experience troubleshooting Group Policy processing and security filtering.

---

### TLS / Certificate Troubleshooting

Uptime Kuma initially reported:

```text
unable to get local issuer certificate
```

The troubleshooting process involved isolating each layer:

```text
DNS
↓
Network connectivity
↓
Caddy
↓
TLS certificate
↓
Certificate authority
↓
Container CA store
↓
Node.js certificate trust
```

The Caddy root certificate was installed into the Ubuntu host's trusted CA store and then mounted into the Uptime Kuma container.

Because Uptime Kuma runs on Node.js, `NODE_EXTRA_CA_CERTS` was configured so the application itself trusted the Caddy internal CA.

The final result allowed Uptime Kuma to monitor HTTPS services without disabling TLS verification.

---

### Active Directory Account Recovery

An account recovery exercise was performed using the domain user account.

The workflow included:

1. Disabling the account
2. Testing authentication failure
3. Resetting the password
4. Re-enabling the account
5. Verifying authentication with the new password

This also demonstrated the difference between cached credentials and fresh domain authentication.

---

## Skills Demonstrated

### Systems Administration

* Linux administration
* Ubuntu Server
* Windows Server
* Windows 11
* SSH
* User and permission management
* Service management
* Certificate management

### Virtualization

* Proxmox VE
* Virtual machines
* LXC containers
* VM networking
* VirtIO devices
* Static addressing
* Resource allocation

### Networking

* IPv4
* Subnetting
* Gateways
* DNS
* DHCP reservations
* Static IP configuration
* Routing
* VPNs
* TCP/UDP ports
* Reverse proxies

### Docker

* Docker Engine
* Docker Compose
* Containers
* Images
* Volumes
* Bind mounts
* Container networking
* Restart policies
* Port mappings
* Portainer

### Windows / Enterprise Administration

* Active Directory Domain Services
* DNS
* Organizational Units
* Security groups
* Domain users
* Domain joining
* Domain authentication
* Group Policy
* Group Policy Preferences
* Security filtering
* Local administrator delegation
* `gpupdate`
* `gpresult`

### Security / Infrastructure

* Tailscale VPN
* Internal PKI
* TLS certificates
* Certificate authorities
* HTTPS
* DNS filtering
* Least-privilege concepts
* Local administrator delegation
* Container isolation

### Troubleshooting

* DNS troubleshooting
* Group Policy troubleshooting
* Authentication troubleshooting
* TLS/SSL troubleshooting
* Certificate trust
* Container-level troubleshooting
* Network connectivity testing
* Layered fault isolation

---

## Repository Contents

### [`documentation/`](documentation/)

Contains the complete technical documentation for the project, including architecture, configuration, deployment procedures, troubleshooting, and lessons learned.

### [`diagrams/`](diagrams/)

Contains architecture diagrams for the primary Linux/Docker homelab and Windows/Active Directory lab.

### [`docker/`](docker/)

Contains Docker Compose configurations and application source/configuration used to deploy the Docker services.

### [`screenshots/`](screenshots/)

Contains selected screenshots documenting the completed infrastructure and configuration.

---

## Full Documentation

For a detailed record of the project, including configuration steps, troubleshooting procedures, networking concepts, and implementation details, see:

**[Full Technical Documentation](documentation/Homelab-Technical-Documentation.pdf)**

The Markdown version is also available in the `documentation/` directory.

---

## Project Status

The core homelab infrastructure is operational.

### Completed

* [x] Proxmox virtualization host
* [x] Ubuntu Docker host
* [x] Docker Engine
* [x] Docker Compose
* [x] Portainer
* [x] Jellyfin
* [x] Navidrome
* [x] Nextcloud
* [x] Kavita
* [x] Caddy reverse proxy
* [x] Internal HTTPS / TLS
* [x] Pi-hole DNS
* [x] Tailscale subnet routing
* [x] Uptime Kuma monitoring
* [x] Custom homelab dashboard
* [x] Windows Server domain controller
* [x] Active Directory
* [x] Windows 11 domain workstation
* [x] Group Policy
* [x] AD account troubleshooting
* [x] DNS and network troubleshooting
* [x] TLS/certificate troubleshooting

The homelab remains an ongoing project and will continue to be expanded as new technologies and administration concepts are explored.
