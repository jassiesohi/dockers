# Docker Engine Installation Guide - Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4 - Same Steps for all three Linux Distributions
## Environment Setup

# Prerequisites
- VMware Workstation Pro 17.6.1
- Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4 ISO
- Basic knowledge of Linux and Docker concepts

# System Requirements
- Minimum 2 GB RAM (4 GB recommended)
- 200 GB disk space (think provision 200 GB disk space for Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4)
- 2 CPU cores
- Internet connection

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
1. VMware virtual machine setup
2. Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4 installation
3. Docker Engine installation and configuration
4. Basic Docker operations verification

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*



# To install Docker Engine on Rocky Linux 9.4, follow these step-by-step commands:

### Step 1: Update Your System

# First, ensure your system is up to date:


sudo dnf update -y


### Step 2: Remove Conflicting Packages

# If you have `podman` or `buildah` installed, remove them to avoid conflicts:


sudo dnf remove podman buildah -y


### Step 3: Add Docker Repository

# Add the official Docker repository to your package manager:


sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo


### Step 4: Install Docker

# Now, install Docker using the following command:


sudo dnf install -y docker-ce


# If you encounter any issues with dependencies, you can use the following command instead:


sudo dnf install docker-ce --allowerasing -y


### Step 5: Start and Enable Docker Service

# After installation, start the Docker service and enable it to run at boot:


sudo systemctl start docker
sudo systemctl enable docker




### Step 6: Verify Docker Status

Check the status of the Docker service to ensure it is running:


sudo systemctl status docker


### Step 7: Add Your User to the Docker Group (Optional)

To allow your user to run Docker commands without `sudo`, add your user to the `docker` group:


sudo usermod -aG docker $USER


# Then, either log out and back in or run the following command to apply the changes:


newgrp docker


### Step 8: Test Docker Installation

Run a test container to verify that Docker is installed correctly:


docker run hello-world


If everything is set up correctly, you should see a message confirming that Docker is working.

### Summary of Commands

Here’s a summary of all commands you'll need:


# Update system packages
sudo dnf update -y

# Remove conflicting packages (if any)
sudo dnf remove podman buildah -y

# Add Docker repository
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo dnf install -y docker-ce  # or use --allowerasing if needed

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Check status of Docker service
sudo systemctl status docker

# Optional: Add user to the Docker group for non-sudo access
sudo usermod -aG docker $USER
newgrp docker  # Apply group changes

# Test Docker installation with hello-world container
docker run hello-world

# List the Namespaces

lsns


# List the PID Namespaces

lsns -t pid


# Launching Multiple Containers

for i in {1..10}; do docker run -d nginx:latest; done


# List the running containers

docker ps


# Stop all containers

docker stop $(docker ps -aq)

docker ps -a
docker ps -aq

# Remove all containers

docker rm $(docker ps -aq)


# Edit the .bashrc file

vi ~/.bashrc
alias dk='docker' # Docker command
alias dkps='docker ps -a' # List all containers
alias dkpsq='docker ps -aq' # List all container IDs
alias dkstop='docker stop $(docker ps -aq)' # Stop all containers
alias dkrm='docker rm $(docker ps -aq)' # Remove all containers
alias dkrmi='docker rmi $(docker images -aq)' # Remove all images

source ~/.bashrc

# Deploy removable containers

docker run --rm -d --name my-nginx nginx:latest

docker ps

docker ps -a

docker stop my-nginx

docker ps -a


# Deploy a container with port forwarding

docker run --rm -d --name my-nginx -p 8000:80 nginx:latest

docker ps

curl http://localhost:8000

# Deploy a busybox container

docker run --rm -d --name my-busybox busybox sleep 3600

docker ps

docker exec -it my-busybox sh

docker exec -it my-busybox ping google.com


# Inspect a container: 
docker inspect my-busybox


# View Docker Container logs
docker logs my-busybox -f

# List networks
docker network ls

# Check the IP address of the host
ip addr show

# Basic ipvlan network creation
docker network create --driver ipvlan \
--subnet=192.168.100.0/24 \
--gateway=192.168.100.1 \
-o ipvlan_mode=l2 \
-o parent=ens160 my-ipvlan-network

# List networks
docker network ls

# Inspect the created network
docker network inspect my-ipvlan-network

# Run a test container on the ipvlan network
docker run --rm -d --name my-nginx --network my-ipvlan-network nginx

# Inspect the container
docker inspect my-nginx

"Networks": {
                "my-ipvlan-network": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "MacAddress": "",
                    "DriverOpts": null,
                    "NetworkID": "d891d1b632bc33e3f78e75cb8adfc37440a9bd0e36efbc8bc2998a5c6e861a88",
                    "EndpointID": "ad16aab6a6fbbfe34dfe47fc89ed7327e2083fb8e60942d79771793b5644ec43",
                    "Gateway": "192.168.100.1",
                    "IPAddress": "192.168.100.2",
                    "IPPrefixLen": 24,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "DNSNames": [
                        "my-nginx",
                        "2c865ab616ad"
                    ]
                }
            }




