# Copilot Instructions for srv-qlever

## Repository Overview

This repository contains Docker Swarm stack configuration files for deploying:

- **QLever** (`docker-stack.yml`) – A high-performance SPARQL graph database with a web UI
- **Portainer** (`docker-compose-portainer.yml`) – A container management platform for Docker Swarm

The primary language used is **YAML** (Docker Compose / Docker Stack format).

## Key Files

| File | Purpose |
|------|---------|
| `docker-stack.yml` | Main QLever stack (qlever-control + qlever-ui services) |
| `docker-compose-portainer.yml` | Portainer CE + Agent stack |
| `.env.example` | Template for environment variable configuration |
| `QUICKSTART.md` | Quick-start deployment guide |
| `PORTAINER_DEPLOYMENT.md` | Portainer-specific deployment guide |
| `.github/workflows/validate-stack.yml` | CI workflow that validates the Docker stack YAML |

## Architecture

Both stacks are designed for **Docker Swarm** (not standalone Docker Compose). They use overlay networks, placement constraints, and replicated/global service modes appropriate for swarm deployments.

## Development Guidelines

### Docker Stack YAML

- Use `version: '3.8'` or higher for `docker-stack.yml`; lower versions are acceptable for Portainer stacks that require legacy syntax
- Always define `deploy.restart_policy` for production services
- Define resource `limits` and `reservations` under `deploy.resources` for resource-intensive services
- Prefer pinned image tags over `:latest` in production; `:latest` is acceptable for development/staging configurations
- Use `placement.constraints` to control which nodes run each service (e.g., manager-only for stateful services)
- Document any security-sensitive volume mounts (e.g., `/var/run/docker.sock`) with inline comments explaining the risk

### Environment Variables

- All configurable values should be listed in `.env.example` with sensible defaults and inline comments
- Never commit a `.env` file with real values – it is listed in `.gitignore`

### Security

- Mounting `/var/run/docker.sock` grants the container root-equivalent access to the Docker daemon; always add a warning comment when this is required
- Avoid exposing management ports (e.g., Portainer 9000/9443) publicly without a reverse proxy and TLS termination

## Validation

The CI workflow (`.github/workflows/validate-stack.yml`) validates `docker-stack.yml` on every push and pull request to `main`/`master`. It checks:

1. Docker stack config syntax (`docker stack config -c docker-stack.yml`)
2. YAML syntax (Python `yaml.safe_load`)
3. Common issues (`:latest` tags, resource limits, health checks)
4. Presence of `.env.example`

When modifying `docker-stack.yml`, ensure the stack passes all CI checks before merging.

## Testing Changes Locally

```bash
# Initialize a local swarm (if not already done)
docker swarm init

# Validate stack syntax
docker stack config -c docker-stack.yml

# Deploy the QLever stack
docker stack deploy -c docker-stack.yml qlever

# Check service status
docker stack services qlever

# Remove the stack when done testing
docker stack rm qlever
```
