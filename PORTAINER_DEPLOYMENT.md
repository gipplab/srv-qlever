# Deploying QLever Stack with Portainer

This guide provides step-by-step instructions for deploying the QLever stack using Portainer.

## Prerequisites

1. Docker Swarm cluster initialized
2. Portainer CE or EE installed and running
3. Access to Portainer web interface
4. Data directories created on the host(s) where you want to run QLever

## Deployment Steps

### 1. Prepare the Host System

SSH into your Docker Swarm manager node and create the required directories:

```bash
sudo mkdir -p /data/qlever/index
sudo mkdir -p /data/qlever/cache
sudo chown -R 1000:1000 /data/qlever  # Adjust UID/GID as needed
sudo chmod -R 755 /data/qlever
```

### 2. Access Portainer

1. Open your web browser and navigate to your Portainer instance (e.g., `https://portainer.example.com`)
2. Log in with your credentials
3. Select your Docker Swarm endpoint

### 3. Deploy the Stack via Portainer UI

#### Option A: Upload the Stack File

1. In the left sidebar, click on **Stacks**
2. Click the **+ Add stack** button
3. Enter a name for your stack (e.g., `qlever-production`)
4. Choose **Upload** tab
5. Click **Select file** and choose `docker-stack.yml` from this repository
6. (Optional) Under **Environment variables**, click **+ add an environment variable** to override any default values
7. Scroll down and click **Deploy the stack**

#### Option B: Use Git Repository

1. In the left sidebar, click on **Stacks**
2. Click the **+ Add stack** button
3. Enter a name for your stack (e.g., `qlever-production`)
4. Choose **Repository** tab
5. Enter repository details:
   - **Repository URL**: `https://github.com/YOUR_USERNAME/srv-qlever` (replace with your fork/repo URL)
   - **Repository reference**: `main` (or your preferred branch)
   - **Compose path**: `docker-stack.yml`
6. (Optional) Configure authentication if using a private repository
7. (Optional) Enable automatic updates
8. Scroll down and click **Deploy the stack**

#### Option C: Web Editor

1. In the left sidebar, click on **Stacks**
2. Click the **+ Add stack** button
3. Enter a name for your stack (e.g., `qlever-production`)
4. Choose **Web editor** tab
5. Copy and paste the contents of `docker-stack.yml`
6. (Optional) Modify the configuration directly in the editor
7. (Optional) Add environment variables
8. Scroll down and click **Deploy the stack**

### 4. Monitor Deployment

After clicking deploy:

1. Portainer will show the deployment progress
2. Once deployed, you'll be redirected to the stack details page
3. You can see the services, networks, and volumes created

### 5. Verify Services are Running

1. In the stack details page, check the **Services** section
2. Both `qlever` and `qlever-ui` services should show as running
3. Click on a service name to view:
   - Container details
   - Logs
   - Stats
   - Service configuration

### 6. View Logs

To view service logs through Portainer:

1. Navigate to **Stacks** → your stack name
2. Click on the service name (e.g., `qlever_qlever`)
3. Click the **Logs** tab
4. You can:
   - View live logs
   - Search logs
   - Download logs
   - Adjust log display options

### 7. Access the Application

Once deployed and running:

- **SPARQL Endpoint**: `http://your-server-ip:7001`
- **Web UI**: `http://your-server-ip:7000`

Replace `your-server-ip` with the actual IP address or domain name of your Docker Swarm node.

## Managing the Stack

### Updating the Stack

1. Navigate to **Stacks** → your stack name
2. Click the **Editor** tab
3. Modify the stack configuration
4. Click **Update the stack**
5. Portainer will perform a rolling update

Alternatively, if using Git repository method:
1. Push your changes to the Git repository
2. Navigate to your stack in Portainer
3. Click **Pull and redeploy** to update from Git

### Scaling Services

1. Navigate to **Stacks** → your stack name
2. Click on the service you want to scale
3. Look for the **Replicas** section
4. Adjust the number of replicas
5. Click **Update the service**

**Note**: QLever is typically run with 1 replica due to its stateful nature.

### Restarting Services

1. Navigate to the service detail page
2. Click the **Update the service** button
3. Select **Force update**
4. Click **Update**

### Viewing Service Metrics

1. Navigate to the service detail page
2. Click the **Stats** tab
3. View real-time CPU, memory, and network usage

### Managing Volumes

1. In the left sidebar, click **Volumes**
2. Filter by stack name to see stack-related volumes
3. Click on a volume to:
   - View usage statistics
   - Browse volume contents (if supported)
   - Remove the volume (when stack is removed)

## Troubleshooting in Portainer

### Checking Service Status

If a service is not running:

1. Navigate to the service
2. Check the **Status** field
3. Click **Logs** to see error messages
4. Check **Events** for deployment issues

### Inspecting Containers

1. Navigate to **Containers** in the sidebar
2. Filter by your stack name
3. Click on a container to:
   - View detailed stats
   - Access the console
   - Inspect container configuration
   - View logs

### Accessing Container Console

For debugging:

1. Navigate to the container detail page
2. Click the **Console** button
3. Select `/bin/bash` or `/bin/sh`
4. Click **Connect**
5. You'll have a terminal inside the container

### Common Issues

#### Service Fails to Start

1. Check logs for error messages
2. Verify data directories exist: `/data/qlever/index` and `/data/qlever/cache`
3. Check resource constraints (CPU/Memory limits)
4. Verify no port conflicts exist

#### Volume Mount Issues

1. Ensure the host paths exist before deployment
2. Check directory permissions
3. If using node placement, ensure paths exist on the correct node

#### Network Issues

1. Verify the overlay network is created
2. Check that services are attached to the correct network
3. Use the console to ping between services

## Advanced Portainer Features

### Using Secrets

For sensitive data:

1. Navigate to **Secrets** in the sidebar
2. Create a new secret
3. Reference it in your stack file:

```yaml
secrets:
  - qlever_config

services:
  qlever:
    secrets:
      - qlever_config
```

### Using Configs

For configuration files:

1. Navigate to **Configs** in the sidebar
2. Create a new config
3. Reference it in your stack file:

```yaml
configs:
  - source: qlever_settings
    target: /etc/qlever/settings.conf
```

### Health Checks

Monitor service health through Portainer's service dashboard, which automatically displays health check status if configured in your stack file.

### Scheduling Updates

If using Portainer Business Edition:

1. Navigate to your stack
2. Configure automatic updates from Git
3. Set up webhooks for triggered updates
4. Schedule updates during maintenance windows

## Best Practices

1. **Use Git Repository Method**: Enables version control and easier updates
2. **Tag Images**: Don't use `latest` in production; use specific version tags
3. **Enable Auto-updates**: Let Portainer automatically pull and deploy updates
4. **Set Resource Limits**: Prevent resource exhaustion
5. **Monitor Logs**: Regularly check logs for errors or warnings
6. **Backup Volumes**: Regularly backup `/data/qlever/index`
7. **Use Secrets**: For sensitive configuration data
8. **Test in Staging**: Test stack updates in a staging environment first

## Additional Resources

- [Portainer Documentation](https://docs.portainer.io/)
- [Docker Stack Deployment](https://docs.docker.com/engine/swarm/stack-deploy/)
- [QLever Documentation](https://docs.qlever.dev/)

## Support

For issues with:
- **Portainer**: See [Portainer Support](https://www.portainer.io/support)
- **QLever**: See [QLever GitHub Issues](https://github.com/ad-freiburg/qlever/issues)
- **This Configuration**: Open an issue in this repository
