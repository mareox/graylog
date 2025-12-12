# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the Docker Compose configuration for a Graylog 6 logging server deployment. The setup is based on the Lawrence Systems Graylog template (https://github.com/lawrencesystems/graylog) and is designed to be deployed via Portainer's Git-based stack deployment feature for automatic updates.

## Architecture

The Graylog stack consists of three primary services:

1. **MongoDB** - Stores Graylog configuration and metadata
2. **OpenSearch** - Backend for log storage and search indexing
3. **Graylog** - Central logging platform with web interface and log ingestion

All services run in a dedicated Docker network and use persistent volumes to preserve data across container restarts.

## Deployment Model

- **Primary deployment method**: Portainer Stacks with Git repository integration
- **Server**: graylog.mareoxlan.local (192.168.20.240)
- **SSH access**: User `mareox` with private key authentication
- **Update mechanism**: Portainer automatically pulls changes from this Git repository

## Environment Configuration

Sensitive configuration is stored in `.env` file (gitignored). The .env file contains:
- Server connection details (hostname, ports, SSH credentials)
- Portainer access credentials
- Service-specific ports (Graylog web UI, OpenSearch)

All secret values (passwords, tokens, API keys) must be externalized to `.env` and never committed to Git.

## Key Configuration Files

- `docker-compose.yml` - Main service definitions and orchestration
- `.env` - Environment variables and secrets (not in Git)
- `.gitignore` - Ensures sensitive files are excluded from version control
- `config/graylog/` - Graylog configuration files (if needed for custom configs)

## Working with This Repository

**To update the deployment:**
1. Make changes to `docker-compose.yml` or configuration files
2. Commit and push to Git repository
3. Portainer will automatically detect changes and offer to update the stack
4. Review changes in Portainer before applying

**To access the server:**
```bash
ssh mareox@graylog.mareoxlan.local
# or
ssh mareox@192.168.20.240
```

**Important deployment constraints:**
- Data volumes must be preserved during updates (existing logs and configuration)
- Service names should remain consistent to maintain volume mappings
- Port mappings should align with existing firewall rules and client configurations

## Security Considerations

- The Lawrence Systems template disables OpenSearch security plugins (not production-ready)
- All secrets must be in `.env`, never in docker-compose.yml or Git
- SSH key authentication is configured for server access
- Portainer credentials are managed via .env file

## Service Access Points

- Graylog Web UI: http://graylog.mareoxlan.local:9000
- Syslog Input: TCP/UDP 1514
- GELF Input: TCP/UDP 12201
- OpenSearch API: Port 9200 (internal)
- Portainer UI: https://graylog.mareoxlan.local:9443
