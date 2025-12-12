# Graylog 6 Docker Deployment

Production-ready Graylog 6 deployment for centralized log management, configured for Portainer Git-based stack deployment.

## Architecture

This deployment consists of three containerized services:

- **MongoDB 6.0.5** - Stores Graylog configuration and metadata
- **OpenSearch 2.x** - Backend for log storage and search indexing
- **Graylog 6.3** - Central logging platform with web interface

All services run on a dedicated Docker bridge network (`graynet`) with persistent volumes for data retention.

## Prerequisites

- Docker Engine 20.10+
- Docker Compose v2+
- Portainer (for Git-based deployment)
- At least 4GB RAM (2GB+ for OpenSearch)
- Firewall rules configured for required ports

## Initial Setup

### 1. Configure Environment Variables

Copy the example environment file and customize it:

```bash
cp .env.example .env
nano .env
```

**Required changes:**
- `GRAYLOG_PASSWORD_SECRET` - Generate a secure 16+ character string
- `GRAYLOG_ROOT_PASSWORD_SHA2` - Generate with: `echo -n "yourpassword" | sha256sum`
- `OPENSEARCH_ADMIN_PASSWORD` - Set a secure password (letters + numbers required)

**Optional configurations:**
- Email/SMTP settings for alerts
- Timezone (default: America/Detroit)
- Custom ports

### 2. Deploy via Portainer

#### Method 1: Git Repository Stack (Recommended)

1. In Portainer, navigate to **Stacks** → **Add stack**
2. Select **Repository** as the build method
3. Configure:
   - **Repository URL**: `https://github.com/mareox/graylog`
   - **Repository reference**: `master`
   - **Compose path**: `docker-compose.yml`
4. Enable **Automatic updates** (optional)
5. Add environment variables from your `.env` file
6. Click **Deploy the stack**

#### Method 2: Manual Docker Compose

```bash
# Clone repository
git clone https://github.com/mareox/graylog.git
cd graylog

# Configure .env file
cp .env.example .env
nano .env

# Deploy stack
docker compose up -d

# Check logs
docker compose logs -f
```

## Access

- **Graylog Web UI**: http://graylog.mareoxlan.local:9000
- **Default credentials**: admin / admin (change via `GRAYLOG_ROOT_PASSWORD_SHA2`)
- **OpenSearch API**: http://graylog.mareoxlan.local:9200 (internal use)

## Log Input Ports

Configure these inputs in Graylog UI after deployment:

- **Syslog**: TCP/UDP 1514
- **GELF**: TCP/UDP 12201

## Data Persistence

All data is stored in Docker volumes:

- `mongo_data` - MongoDB configuration database
- `log_data` - OpenSearch indices and logs
- `graylog_data` - Graylog data directory

**Backup recommendation**: Regular backups of these volumes ensure data retention.

## Security Considerations

⚠️ **Important Security Notes:**

1. **OpenSearch security is DISABLED** - This configuration disables OpenSearch security plugins for simplified deployment. For production environments exposed to untrusted networks, consider:
   - Enabling OpenSearch security plugins
   - Implementing network isolation
   - Using reverse proxy with authentication

2. **Change default passwords** - Update all default credentials in `.env`:
   - Graylog root password
   - Graylog password secret
   - OpenSearch admin password

3. **Environment file security** - The `.env` file contains sensitive credentials and is excluded from Git via `.gitignore`. Never commit this file.

4. **Network exposure** - Limit access to Graylog web UI and input ports via firewall rules

## Updating the Deployment

### Via Portainer (Automatic)

If automatic updates are enabled in Portainer:
1. Push changes to the GitHub repository
2. Portainer will detect updates and prompt for redeployment
3. Review changes and confirm update

### Manual Update

```bash
# Pull latest changes
git pull origin master

# Restart stack (preserves data)
docker compose up -d

# View updated services
docker compose ps
```

## Maintenance Commands

```bash
# View logs
docker compose logs -f graylog
docker compose logs -f opensearch

# Restart services
docker compose restart graylog

# Stop all services
docker compose down

# Stop and remove volumes (DELETES DATA!)
docker compose down -v

# Check service status
docker compose ps

# View resource usage
docker stats
```

## Troubleshooting

### OpenSearch fails to start

Check memory limits and ensure vm.max_map_count is set:

```bash
# On Docker host
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

### Graylog web UI not accessible

1. Check if container is running: `docker compose ps`
2. Verify port binding: `docker compose port graylog 9000`
3. Check logs: `docker compose logs graylog`
4. Verify `GRAYLOG_HTTP_EXTERNAL_URI` in `.env`

### Data persistence issues

Ensure volumes are properly mounted:

```bash
docker volume ls
docker volume inspect graylog_mongo_data
docker volume inspect graylog_log_data
docker volume inspect graylog_graylog_data
```

## Email Alerts Configuration

To enable email notifications:

1. Update `.env` with your SMTP details:
   ```
   GRAYLOG_TRANSPORT_EMAIL_ENABLED=true
   GRAYLOG_TRANSPORT_EMAIL_HOSTNAME=smtp.example.com
   GRAYLOG_TRANSPORT_EMAIL_AUTH_USERNAME=user@example.com
   GRAYLOG_TRANSPORT_EMAIL_AUTH_PASSWORD=yourpassword
   ```

2. Restart Graylog: `docker compose restart graylog`

3. Test email settings in Graylog UI: System → Configurations → Email

## Resources

- [Graylog Documentation](https://docs.graylog.org/)
- [OpenSearch Documentation](https://opensearch.org/docs/)
- [Lawrence Systems Template](https://github.com/lawrencesystems/graylog)

## License

Based on the Lawrence Systems Graylog template. See original repository for license details.
