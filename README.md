# srv-qlever
Server config of qlever

## Portainer Stack

This repository includes a Docker Swarm stack for deploying [Portainer CE](https://www.portainer.io/) with the [Portainer Agent](https://docs.portainer.io/start/agent), a container management platform that provides a web-based UI for managing Docker Swarm environments.

### Prerequisites

- Docker Swarm mode must be initialized on your cluster
- This stack is designed for Docker Swarm (not standalone Docker Compose)

### Usage

To deploy Portainer using the included stack file:

```bash
docker stack deploy -c docker-compose-portainer.yml portainer
```

### Accessing Portainer

Once deployed, you can access Portainer at:
- HTTP: http://localhost:9000
- HTTPS: https://localhost:9443
- Edge Agent: http://localhost:8000

On first launch, you'll need to create an admin account.

### Managing the Stack

Remove the stack:
```bash
docker stack rm portainer
```

View services:
```bash
docker stack services portainer
```

View logs (for a specific service):
```bash
docker service logs portainer_portainer
docker service logs portainer_agent
```

### Stack Configuration

The Portainer stack includes:
- **Portainer Agent**: Deployed globally on all Linux nodes, with host filesystem mounted at `/host` for full environment visibility
- **Portainer CE**: LTS version deployed on manager nodes
- **Persistent Storage**: Volume for configuration and data persistence
- **Network**: Overlay network for agent-to-portainer communication
- **Ports**: 9000 (HTTP), 9443 (HTTPS), and 8000 (Edge Agent)
- **Agent Communication**: TCP connection between Portainer and agents via port 9001
