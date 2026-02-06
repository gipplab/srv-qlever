# srv-qlever
Server config of qlever

## Portainer Stack

This repository includes a Docker Compose stack for deploying [Portainer CE](https://www.portainer.io/), a container management platform that provides a web-based UI for managing Docker environments.

### Usage

To deploy Portainer using the included stack file:

```bash
docker compose -f docker-compose-portainer.yml up -d
```

### Accessing Portainer

Once deployed, you can access Portainer at:
- HTTP: http://localhost:9000
- HTTPS: https://localhost:9443

On first launch, you'll need to create an admin account.

### Managing the Stack

Stop Portainer:
```bash
docker compose -f docker-compose-portainer.yml down
```

View logs:
```bash
docker compose -f docker-compose-portainer.yml logs -f
```

### Stack Configuration

The Portainer stack includes:
- **Portainer CE**: Latest community edition
- **Persistent Storage**: Volume for configuration and data persistence
- **Security**: Read-only Docker socket access with no-new-privileges flag
- **Ports**: 9000 (HTTP) and 9443 (HTTPS)
- **Auto-restart**: Configured to restart unless stopped manually
