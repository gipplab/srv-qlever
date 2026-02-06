# srv-qlever

Server configuration for deploying QLever with Docker Swarm and Portainer.

## Overview

This repository contains Docker Stack configuration files for deploying QLever, a high-performance SPARQL graph database, using Docker Swarm. The configuration is optimized for deployment via Portainer.

## Prerequisites

- Docker Engine 19.03.0+ with Swarm mode enabled
- Portainer CE/EE (optional, but recommended for easier management)
- Sufficient storage space for your RDF datasets
- Minimum 16GB RAM (32GB+ recommended for large datasets)

## Quick Start

### 1. Initialize Docker Swarm

If you haven't already initialized Docker Swarm:

```bash
docker swarm init
```

### 2. Prepare Data Directories

Create the required directories for QLever data and cache:

```bash
sudo mkdir -p /data/qlever/index
sudo mkdir -p /data/qlever/cache
sudo chmod -R 755 /data/qlever
```

### 3. Deploy the Stack

#### Using Docker CLI:

```bash
docker stack deploy -c docker-stack.yml qlever
```

#### Using Portainer:

1. Log in to your Portainer instance
2. Navigate to **Stacks** in the sidebar
3. Click **Add stack**
4. Choose **Upload** and select `docker-stack.yml` or paste its contents
5. Name your stack (e.g., "qlever")
6. Click **Deploy the stack**

### 4. Create Your Index

Before QLever can serve queries, you need to create an index. You can do this by:

1. Installing the QLever CLI: `pip install qlever`
2. Creating a Qleverfile for your dataset
3. Running the index creation commands

Alternatively, you can exec into the container and manually create the index.

## Services

### qlever (SPARQL Endpoint)

- **Port**: 7001
- **Purpose**: Main SPARQL query endpoint
- **Image**: adfreiburg/qlever:latest
- **Resources**: 
  - Limits: 4 CPUs, 32GB RAM
  - Reservations: 2 CPUs, 16GB RAM

### qlever-ui (Web Interface)

- **Port**: 7000
- **Purpose**: Web-based query interface with autocompletion
- **Image**: adfreiburg/qlever-ui:latest
- **Backend**: Connected to qlever service

## Configuration

### Environment Variables

You can customize the deployment by modifying environment variables in `docker-stack.yml`:

- `QLEVER_PORT`: Port for the SPARQL endpoint (default: 7001)
- `QLEVER_MEMORY_FOR_QUERIES`: Memory allocated for query processing (default: 30G)
- `QLEVER_CACHE_MAX_SIZE`: Maximum cache size (default: 10G)
- `QLEVER_BACKEND_URL`: URL for the UI to connect to the backend (default: http://qlever:7001)

### Volumes

- `qlever_data`: Stores the RDF index (mounted to `/data/qlever/index`)
- `qlever_cache`: Stores query cache (mounted to `/data/qlever/cache`)

**Note**: The volumes are configured as bind mounts. Make sure the directories exist before deploying.

### Resource Limits

Adjust CPU and memory limits in the `deploy.resources` section based on your:
- Dataset size
- Expected query load
- Available server resources

## Accessing the Services

After deployment:

- **SPARQL Endpoint**: http://your-server:7001
- **Web UI**: http://your-server:7000

## Monitoring

Check the status of your stack:

```bash
# List services
docker stack services qlever

# View service logs
docker service logs qlever_qlever
docker service logs qlever_qlever-ui

# Check service details
docker service inspect qlever_qlever
```

## Updating the Stack

To update the stack configuration:

```bash
docker stack deploy -c docker-stack.yml qlever
```

Docker Swarm will perform a rolling update based on the update configuration.

## Scaling

To scale the QLever service (if needed):

```bash
docker service scale qlever_qlever=2
```

**Note**: Most QLever deployments use a single replica due to the stateful nature of the index.

## Removing the Stack

To completely remove the stack:

```bash
docker stack rm qlever
```

**Warning**: This does not remove the volumes. To remove volumes, use:

```bash
docker volume rm qlever_qlever_data qlever_qlever_cache
```

## Troubleshooting

### Service won't start

1. Check logs: `docker service logs qlever_qlever`
2. Verify directories exist: `/data/qlever/index` and `/data/qlever/cache`
3. Ensure sufficient resources are available

### No index found message

The QLever service requires an index to be created before it can serve queries. Follow the index creation steps in the Quick Start section.

### Port conflicts

If ports 7000 or 7001 are already in use, modify the port mappings in `docker-stack.yml`:

```yaml
ports:
  - "8001:7001"  # Change external port
```

## Advanced Configuration

For production deployments, consider:

1. **Persistent Storage**: Use NFS or other distributed storage for volumes
2. **Load Balancing**: Configure ingress load balancing for multiple replicas
3. **Secrets Management**: Use Docker secrets for sensitive configuration
4. **Health Checks**: Add health check endpoints for better monitoring
5. **Backup Strategy**: Regular backups of `/data/qlever/index`

## Additional Resources

- [QLever Main Repository](https://github.com/ad-freiburg/qlever)
- [QLever CLI Documentation](https://github.com/ad-freiburg/qlever-control)
- [QLever Documentation](https://docs.qlever.dev/)
- [Docker Stack Documentation](https://docs.docker.com/engine/reference/commandline/stack/)
- [Portainer Documentation](https://docs.portainer.io/)

## License

This configuration is provided as-is. QLever itself is licensed under Apache 2.0.
