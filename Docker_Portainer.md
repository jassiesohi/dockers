
# Docker Portainer Community Edition Setup on macvlan network on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4

## Environment Setup
# Prerequisites
- VMware Workstation Pro 17.6.1
- Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4 VM
- Basic knowledge of Linux and Docker concepts

# System Requirements
- Minimum 2 GB RAM (4 GB recommended)
- 200 GB disk space (thin provision 200 GB disk space for Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4)
- 2 CPU cores
- Internet connection
- Docker Engine installed on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4
# About This Guide
This hands-on tutorial is designed for practitioners who:
- Have basic understanding of Linux and Docker
- Want practical, step-by-step instructions
- Prefer a no-frills approach to learning
- Need a reliable reference for Docker setup

# Target Audience
- DevOps beginners
- System administrators
- Developers starting with containerization
- IT students with basic Linux knowledge

# What's Covered
1. Docker Portainer Setup on macvlan network on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*


# When using macvlan networking, port mapping (-p) works differently than with the default bridge network. 
# Let me explain how to modify the Portainer setup for macvlan.


# Key differences and important points:

# Port Mapping with Macvlan:
# The -p flags are removed because port mapping doesn't work with macvlan
# Instead, the container gets a direct IP address on your network (192.168.100.10 in this example)
# You'll access Portainer directly via its IP: https://192.168.100.10:9443


# Network Access:
# The container behaves like a physical device on your network
# Ports 8000 and 9443 are automatically exposed on the container's IP
# No port mapping is needed as the container has its own IP address


# Considerations:
# Make sure 192.168.100.10 is not used by another device
# Your host machine needs to be able to route to the macvlan network
# You might need to configure your firewall to allow access to ports 9443 and 8000
# The container will be directly accessible to other devices on your network


# To access Portainer:
# Web UI: https://192.168.100.10:9443
# API: http://192.168.100.10:8000


# 1. First create the macvlan network
docker network create -d macvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=ens160 macvlan_net

# 2. Run Portainer Community Edition with macvlan network and static IP
docker run -d \
  --name portainer \
  --network macvlan_net \
  --ip=192.168.100.10 \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# To verify the container's IP and network settings
docker inspect portainer | grep IPAddress

# If you need to remove and recreate:
docker rm -f portainer
docker volume rm portainer_data

# Inspect the portainer container
docker inspect portainer

# Remove the portainer container
docker rm -f portainer

# Remove the portainer data volume
docker volume rm portainer_data 


# Docker will automatically create the named volume portainer_data if it doesn't already exist when you run the container.
# You can verify this in a few ways:
docker volume ls
docker volume inspect portainer_data

# List all Docker volumes
docker volume ls | grep portainer_data

# Inspect the volume details
docker volume inspect portainer_data

# The volume data is typically stored in
ls -l /var/lib/docker/volumes/portainer_data/_data

# in my case the volume data is stored in /var/lib/docker/volumes/portainer_data/_data


# If you want to create the volume explicitly before running the container, you can do
docker volume create portainer_data

# The key differences between letting Docker create it automatically vs creating it manually are:

# Auto-creation is more convenient
# Both methods result in the same outcome
# The volume persists even if you remove the container
# If you remove the volume (docker volume rm portainer_data), all Portainer settings will be lost

# Remember: If you need to completely reset Portainer, you'd need to

# docker rm -f portainer          # Remove container
# docker volume rm portainer_data # Remove volume



# Thanks for Watching!