
Here are the commands for working with Nginx containers:

1. Basic Nginx container creation:

# Pull and run Nginx with port mapping
docker run -d --name my_nginx -p 80:80 nginx

# With specific name and port mapping (e.g., host port 8080)
docker run -d --name my_nginx -p 8080:80 nginx


2. Run with volume mounting for custom configuration:

# Mount custom config directory
docker run -d --name my_nginx \
    -p 80:80 \
    -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
    -v $(pwd)/html:/usr/share/nginx/html:ro \
    nginx


3. Access container bash:

docker exec -it my_nginx bash


4. Install utilities inside container:

# Update package list
apt-get update

# Install common tools
apt-get install -y \
    curl \
    wget \
    vim \
    procps \
    net-tools \
    iputils-ping


5. Common Nginx commands inside container:

# Test Nginx configuration
nginx -t

# Reload Nginx configuration
nginx -s reload

# Stop Nginx
nginx -s stop

# View Nginx version
nginx -v

# View detailed version
nginx -V


6. View logs:

# View access logs
docker logs my_nginx

# Follow logs in real-time
docker logs -f my_nginx

# View logs inside container
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log


7. Common file locations inside container:

# Configuration files
/etc/nginx/nginx.conf          # Main config
/etc/nginx/conf.d/            # Additional configs
/usr/share/nginx/html/        # Default web root
/var/log/nginx/              # Log files


8. Container management:

# Stop container
docker stop my_nginx

# Start container
docker start my_nginx

# Restart container
docker restart my_nginx

# Remove container
docker rm -f my_nginx


9. Running with persistent storage:

# Create volumes
docker volume create nginx_conf
docker volume create nginx_html

# Run with volumes
docker run -d --name my_nginx \
    -p 80:80 \
    -v nginx_conf:/etc/nginx/conf.d \
    -v nginx_html:/usr/share/nginx/html \
    nginx


10. Running with custom network:

# Create network
docker network create nginx_net

# Run container in network
docker run -d --name my_nginx \
    --network nginx_net \
    -p 80:80 \
    nginx


11. Health check:

# Basic health check
docker inspect --format='{{.State.Health.Status}}' my_nginx

# Run with health check
docker run -d --name my_nginx \
    -p 80:80 \
    --health-cmd="curl --silent --fail http://localhost:80 || exit 1" \
    --health-interval=10s \
    --health-timeout=2s \
    --health-retries=3 \
    nginx


Remember:
- Always test configuration changes before applying them
- Back up important data before making changes
- Use volumes for persistent data
- Consider using Docker Compose for more complex setups
- Keep security in mind when exposing ports

