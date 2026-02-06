# Quick Reference Guide

## Quick Commands

### Deploy Stack
```bash
docker stack deploy -c docker-stack.yml qlever
```

### Check Status
```bash
docker stack services qlever
```

### View Logs
```bash
# QLever service logs
docker service logs -f qlever_qlever

# UI service logs
docker service logs -f qlever_qlever-ui
```

### Update Stack
```bash
docker stack deploy -c docker-stack.yml qlever
```

### Remove Stack
```bash
docker stack rm qlever
```

### Scale Service
```bash
docker service scale qlever_qlever=1
```

## Service Endpoints

- **SPARQL Endpoint**: http://localhost:7001
- **Web UI**: http://localhost:7000

## Example SPARQL Query

```sparql
SELECT ?subject ?predicate ?object
WHERE {
  ?subject ?predicate ?object
}
LIMIT 10
```

Access via:
```bash
curl -X POST http://localhost:7001 \
  -H "Content-Type: application/sparql-query" \
  -H "Accept: application/sparql-results+json" \
  -d "SELECT ?s ?p ?o WHERE { ?s ?p ?o } LIMIT 10"
```

## Common Tasks

### Create Data Directory
```bash
sudo mkdir -p /data/qlever/{index,cache}
sudo chmod -R 755 /data/qlever
```

### Check Service Health
```bash
# List all services
docker service ls

# Inspect specific service
docker service inspect qlever_qlever

# View service tasks
docker service ps qlever_qlever
```

### Access Container Shell
```bash
# Find container ID
docker ps | grep qlever

# Exec into container
docker exec -it <container_id> /bin/bash
```

### Monitor Resources
```bash
# Service stats
docker stats $(docker ps -q --filter "label=com.docker.swarm.service.name=qlever_qlever")
```

## Troubleshooting

### Service Not Starting
```bash
# Check logs
docker service logs qlever_qlever

# Check service events
docker service ps qlever_qlever --no-trunc

# Inspect service
docker service inspect qlever_qlever
```

### Port Already in Use
```bash
# Check what's using the port
sudo netstat -tulpn | grep :7001

# Or use ss
sudo ss -tulpn | grep :7001
```

### Volume Issues
```bash
# List volumes
docker volume ls | grep qlever

# Inspect volume
docker volume inspect qlever_qlever_data

# Check mount paths exist
ls -la /data/qlever/
```

## Network Debugging

### Test Service Connectivity
```bash
# From host
curl http://localhost:7001

# From another container in the same network
docker run --rm --network qlever_qlever_network curlimages/curl:latest \
  curl http://qlever:7001
```

### Inspect Network
```bash
docker network inspect qlever_qlever_network
```

## Backup and Restore

### Backup Index Data
```bash
# Create backup
sudo tar -czf qlever-backup-$(date +%Y%m%d).tar.gz /data/qlever/index

# Copy to remote location
rsync -avz /data/qlever/index/ remote-server:/backups/qlever/
```

### Restore Index Data
```bash
# Stop the service first
docker service scale qlever_qlever=0

# Restore data
sudo tar -xzf qlever-backup-YYYYMMDD.tar.gz -C /

# Start the service
docker service scale qlever_qlever=1
```

## Performance Tuning

### Adjust Memory Settings
Edit `docker-stack.yml` and modify:
```yaml
environment:
  - QLEVER_MEMORY_FOR_QUERIES=30G  # Increase for larger datasets
  - QLEVER_CACHE_MAX_SIZE=10G      # Increase for better performance
```

### Adjust Resource Limits
```yaml
resources:
  limits:
    cpus: '8'        # Increase for more CPU power
    memory: 64G      # Increase for larger datasets
```

Then redeploy:
```bash
docker stack deploy -c docker-stack.yml qlever
```

## Security

### Use Docker Secrets
```bash
# Create a secret
echo "my-secret-value" | docker secret create qlever_secret -

# Reference in stack file
secrets:
  - qlever_secret
```

### Restrict Network Access
```bash
# Use firewall rules
sudo ufw allow from 10.0.0.0/8 to any port 7001
```

## Monitoring

### View All Stack Resources
```bash
docker stack ps qlever
```

### Continuous Log Monitoring
```bash
docker service logs -f --tail 100 qlever_qlever
```

### Export Metrics
Consider using Prometheus with the Docker metrics exporter for production monitoring.

## Additional Resources

- Full documentation: README.md
- Portainer guide: PORTAINER_DEPLOYMENT.md
- QLever docs: https://docs.qlever.dev/
- Docker Swarm docs: https://docs.docker.com/engine/swarm/
