# Architecture Diagrams

This directory contains architecture diagrams documenting the major environments within the homelab.

## Ubuntu / Linux Homelab Architecture

The Ubuntu architecture diagram illustrates the primary infrastructure running on the Proxmox host, including:

* Ubuntu Docker VM
* Docker services
* Tailscale subnet routing
* Pi-hole DNS
* Caddy reverse proxy
* Internal application networking
* Remote access flow

The diagram is intended to provide a high-level view of how traffic moves through the infrastructure.

## Windows / Active Directory Architecture

The Windows architecture diagram illustrates the dedicated Windows lab environment, including:

* DC01
* Windows Server 2025
* Active Directory Domain Services
* DNS
* Group Policy
* CLIENT01
* Windows 11
* Domain membership
* Network connectivity

The Windows environment is isolated conceptually as an enterprise administration practice environment while sharing the physical Proxmox infrastructure.

## Diagram Notes

The diagrams were created manually as part of the project documentation.

They are intentionally designed as architecture-level diagrams rather than detailed network topology maps. The complete technical documentation contains additional configuration details and addressing information.
