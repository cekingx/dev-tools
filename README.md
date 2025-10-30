# Development Tools

A modular Docker Compose setup providing essential infrastructure services for local development. Each service is isolated in its own directory with independent compose files for flexible management.

## Prerequisites

- Docker Engine 20.10+
- Docker Compose V2

## Project Structure

```
dev-tools/
├── kafka/          # Message queue with Zookeeper and Kouncil UI
├── mysql/          # MySQL 8.0 database
├── network/        # Docker networks and test container
├── nginx-proxy/    # Nginx Proxy Manager with Let's Encrypt
├── portainer/      # Docker container management UI
├── postgres/       # PostgreSQL 14 database
├── redis/          # Redis cache/data store
├── registry/       # Local Docker image registry
└── sonarqube/      # SonarQube Community Edition code analysis
```

## Services Overview

### Databases

#### MySQL
- **Image**: `mysql:8.0.41`
- **Port**: `3306`
- **Credentials**:
  - Root: `root` / `root13`
  - User: `cekingx` / `root13`
- **Data**: Persisted in `mysql/storage/`
- **Usage**:
  ```bash
  cd mysql && docker compose up -d
  mysql -h 127.0.0.1 -u cekingx -proot13
  ```

#### PostgreSQL
- **Image**: `postgres:14.6`
- **Port**: `5432`
- **Credentials**: `postgres` / `root13`
- **Data**: Persisted in `postgres/storage/`
- **Usage**:
  ```bash
  cd postgres && docker compose up -d
  psql -h localhost -U postgres
  ```

#### Redis
- **Image**: `redis:latest`
- **Port**: `6379`
- **Data**: Persisted in `redis/storage/`
- **Usage**:
  ```bash
  cd redis && docker compose up -d
  redis-cli -h localhost
  ```

### Message Queue

#### Kafka Stack
- **Zookeeper**: Port `2181`
- **Kafka**:
  - External: `localhost:9094`
  - Internal (between services): `kafka-core:9092`
  - Container-to-container: `kafka-core:9093`
- **Kouncil UI**: Port `8080`
  - Web interface for managing Kafka topics and messages
  - Access: http://localhost:8080
- **Networks**: Connected to `backend-network` and `staging-network`
- **Usage**:
  ```bash
  cd kafka && docker compose up -d
  # Access Kouncil at http://localhost:8080
  ```

### Infrastructure Tools

#### Docker Registry
- **Image**: `registry:3`
- **Port**: `5000`
- **Purpose**: Local Docker image registry for private images
- **Usage**:
  ```bash
  cd registry && docker compose up -d
  # Tag and push images
  docker tag myimage:latest localhost:5000/myimage:latest
  docker push localhost:5000/myimage:latest
  ```

#### Portainer
- **Image**: `portainer/portainer-ce:lts`
- **Ports**:
  - `8000` (HTTP)
  - `9443` (HTTPS - Admin UI)
- **Access**: https://localhost:9443
- **Purpose**: Web-based Docker container management
- **Data**: Persisted in `portainer/storage/`
- **Usage**:
  ```bash
  cd portainer && docker compose up -d
  # Access at https://localhost:9443
  # First time: create admin account
  ```

#### Nginx Proxy Manager
- **Image**: `jc21/nginx-proxy-manager:latest`
- **Ports**:
  - `80` (HTTP)
  - `81` (Admin UI)
  - `443` (HTTPS)
- **Access**: http://localhost:81
- **Default Credentials**:
  - Email: `admin@example.com`
  - Password: `changeme` (change on first login)
- **Purpose**: Reverse proxy with Let's Encrypt SSL support
- **Data**:
  - Configuration: `nginx-proxy/storage/`
  - SSL Certificates: `nginx-proxy/letsencrypt/`
- **Usage**:
  ```bash
  cd nginx-proxy && docker compose up -d
  # Access admin UI at http://localhost:81
  ```

### Code Quality & Analysis

#### SonarQube Community Edition
- **Image**: `sonarqube:community`
- **Port**: `9000`
- **Access**: http://localhost:9000
- **Default Credentials**:
  - Username: `admin`
  - Password: `admin` (must change on first login)
- **Database**: PostgreSQL 15 (included in compose file)
  - Internal credentials: `sonarqube` / `sonarqube`
- **Purpose**: Continuous code quality and security analysis
- **Data**: Persisted in `sonarqube/storage/`
  - `sonarqube_data/` - Analysis results and settings
  - `sonarqube_logs/` - Application logs
  - `sonarqube_extensions/` - Plugins and extensions
  - `postgresql_data/` - Database files
- **Usage**:
  ```bash
  cd sonarqube
  # Create storage directories with proper permissions (first time only)
  mkdir -p storage/{sonarqube_data,sonarqube_logs,sonarqube_extensions,postgresql_data}
  chmod -R 777 storage
  # Start services
  docker compose up -d
  # Wait 1-2 minutes for initialization
  # Access at http://localhost:9000
  # First login: admin/admin (change password immediately)
  ```
- **Important Notes**:
  - First startup may take 2-3 minutes to initialize database
  - Requires at least 2GB RAM for optimal performance
  - Storage directories need 777 permissions due to different container users (UID 70 for PostgreSQL, UID 1000 for SonarQube)
  - If you encounter permission errors, run: `docker run --rm -v "$(pwd)/storage:/data" alpine chmod -R 777 /data`
  - Change default admin password immediately after first login
  - Use for analyzing code quality, security vulnerabilities, and code smells

### Networking

#### Network Infrastructure
- **Networks**:
  - `staging-network`: For staging environment services
  - `backend-network`: For backend services
  - `frontend-network`: For frontend services
- **Ubuntu Test Container**:
  - Connected to all networks
  - Useful for network debugging and testing connectivity
- **Usage**:
  ```bash
  cd network && docker compose up -d
  # Enter test container
  docker exec -it ubuntu bash
  ```

## Quick Start

### Start All Services
```bash
# Start network infrastructure first
cd network && docker compose up -d && cd ..

# Start individual services as needed
cd mysql && docker compose up -d && cd ..
cd postgres && docker compose up -d && cd ..
cd redis && docker compose up -d && cd ..
cd kafka && docker compose up -d && cd ..
cd registry && docker compose up -d && cd ..
cd portainer && docker compose up -d && cd ..
cd nginx-proxy && docker compose up -d && cd ..
cd sonarqube && docker compose up -d && cd ..
```

### Start Specific Service
```bash
cd <service-name>
docker compose up -d
```

### Stop Service
```bash
cd <service-name>
docker compose down
```

### View Logs
```bash
cd <service-name>
docker compose logs -f
```

## Network Architecture

Services can be connected to custom networks for isolation:

- **staging-network**: Used by Kafka and test container
- **backend-network**: Used by Kafka and test container
- **frontend-network**: Used by test container

To connect your application to these networks, add to your compose file:
```yaml
networks:
  backend-network:
    external: true
```

## Service URLs

| Service | URL | Purpose |
|---------|-----|---------|
| MySQL | `localhost:3306` | Database connection |
| PostgreSQL | `localhost:5432` | Database connection |
| Redis | `localhost:6379` | Cache connection |
| Kafka | `localhost:9094` | External broker connection |
| Kouncil | http://localhost:8080 | Kafka management UI |
| Portainer | https://localhost:9443 | Container management UI |
| Nginx Proxy Manager | http://localhost:81 | Proxy admin UI |
| Docker Registry | `localhost:5000` | Image registry |
| SonarQube | http://localhost:9000 | Code quality analysis |

## Data Persistence

All service data is stored in local `storage/` directories within each service folder. These directories are git-ignored and contain:

- Database files
- Configuration files
- Application data

To reset a service, stop it and remove its storage directory:
```bash
cd <service-name>
docker compose down
rm -rf storage/
docker compose up -d
```

## Troubleshooting

### Port Conflicts
If ports are already in use, modify the port mapping in the respective `compose.yml`:
```yaml
ports:
  - "NEW_PORT:CONTAINER_PORT"
```

### Network Issues
Ensure networks are created before starting services that depend on them:
```bash
cd network && docker compose up -d
```

### Permission Issues
If you encounter permission errors with volumes:
```bash
sudo chown -R $USER:$USER <service-name>/storage/
```

### View Running Containers
```bash
docker ps
```

### Check Service Health
```bash
cd <service-name>
docker compose ps
docker compose logs
```

## Best Practices

1. **Start networks first**: Always start the `network` service before Kafka
2. **Backup data**: Regularly backup `storage/` directories for important data
3. **Update images**: Periodically pull latest images for security updates
   ```bash
   docker compose pull
   docker compose up -d
   ```
4. **Monitor resources**: Use Portainer to monitor container resource usage
5. **Clean up**: Remove unused containers and images regularly
   ```bash
   docker system prune -a
   ```

## Security Notes

- Change default passwords before exposing services to networks
- The credentials in compose files are for LOCAL DEVELOPMENT ONLY
- Never commit `storage/` directories or `letsencrypt/` certificates
- Use Nginx Proxy Manager for SSL termination in staging/production-like environments

## License

This is a personal development tools setup. Use and modify as needed for your projects.
