
# Docker Bridge, None, Host Setups on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4
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
- Docker Bridge, None, Host Setups on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*



# List Docker Networks
docker network ls

# Inspect the Bridge Network
docker network inspect bridge

# Types of Docker Networks in this tutorial

# Bridge Network
# None Network
# Host Network


# Lets create a User-Defined Bridge Network to have better isolation & enable dns resolution between containers

# Create a User-Defined Bridge Network

docker network create --driver bridge my-custom-bridge-network

docker network ls

docker network inspect my-custom-bridge-network

# Create two busybox containers with custom bridge network & check connectivity between them with ping & dns resolution

docker run --rm -d --name mybusybox --network my-custom-bridge-network busybox sleep 3600
docker run --rm -d --name mybusybox1 --network my-custom-bridge-network busybox sleep 3600


# Inspect the custom bridge network
docker network inspect my-custom-bridge-network

# Inspect the busybox containers
docker inspect mybusybox
docker inspect mybusybox1  


# Ping between busybox containers
docker exec -it mybusybox ping -c 4 172.18.0.3
# Now Test DNS resolution between busybox containers
docker exec -it mybusybox ping -c 4 mybusybox1
docker exec -it mybusybox ping -c 4 google.com

# DNS resolution is now working between busybox containers through the custom bridge network

# Create nginx & ubuntu container with custom bridge network
docker run --rm -d --name mynginx --network my-custom-bridge-network -p 8080:80 nginx

# curl nginx container from host machine
curl http://localhost:8080

# Create ubuntu container with custom bridge network
docker run --rm -it -d --name myubuntu --network my-custom-bridge-network ubuntu sleep infinity

# Install ping utility in ubuntu container
docker exec -it myubuntu apt update
docker exec -it myubuntu apt install -y iputils-ping
docker exec -it myubuntu apt install -y curl

docker exec -it myubuntu ping -c 4 google.com
docker exec -it myubuntu curl mynginx

# Inspect the custom bridge network
docker network inspect my-custom-bridge-network


"Containers": {
            "161b1aa4193a69b2f0876d268bd00979c2075d735db2c72e0b2af24e1f628d27": {
                "Name": "mybusybox",
                "EndpointID": "5381174a01bc1d3cdc7dd815dd9198c326f7d124717c8095fd191d35fc68d06a",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "1b4243c58b73081edd3999765b0c64209bcc9a194641c121739d78f0e38f1da7": {
                "Name": "mynginx",
                "EndpointID": "91be41023e2eef1e4d22a49b50bcc6c13ca38276958fed46f41182fdeee97e95",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "4c98848358fd1d9970424217200d4c0765ca75e5d1954a6bcef7d307fc62b260": {
                "Name": "mybusybox1",
                "EndpointID": "34b524543aa403850d27459c662dfc7751b208241439198bc9788c17f24428f5",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "f718b363288a7ef54beda3960f7369aadffba2c4c6d7196d7d6b1b0ff87c90af": {
                "Name": "myubuntu",
                "EndpointID": "9a4061a3904605516b495b6738f9ff96154326e862f5d04d9aed510350e9c3a9",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            }



# Stop all containers
docker stop $(docker ps -q)

# Delete the custom bridge network
docker network rm my-custom-bridge-network

# List the docker networks
docker network ls

[admin100@rocky-docker-node0 ~]$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
5c2f46a4f0d2   bridge    bridge    local
2b4f3e9063a5   host      host      local
6ba3d1b149c6   none      null      local


# None Network
# None network is used to isolate containers from the network
# None network does not have any network interfaces or DNS resolution

docker run --rm -d --name mybusybox3 --network none busybox sleep 300

docker network inspect none
docker exec mybusybox3 ip addr


[admin100@rocky-docker-node0 ~]$ docker exec mybusybox3 ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever



# Host Network

# Lets create host network
# Host network is used to connect containers directly to the host's network stack
# Containers on host network can directly access the host's network interfaces & services
# Containers on host network can be accessed directly by other hosts on the same network
# it is useful to have a container such as prometheus node exporter which needs to scrape the host's metrics

# lets create a prometheus node exporter container on the host network

# stop all running containers
docker stop $(docker ps -q)

# list the networks
docker network ls

# create prometheus node exporter container on the host network
docker run --rm -d --name my-prometheus-node-exporter --network host prom/node-exporter

docker ps

# check the connectivity from the host to the prometheus node exporter container
curl http://localhost:9100/metrics
curl http://192.168.100.134:9100/metrics

# Seems firewall is blocking the access to the prometheus node exporter container


# Set up the firewall rules in Rocky Linux 9.4 using `firewalld` to allow access to the Node Exporter running on port 9100

# Here are the steps

# 1. First, verify firewalld is running

sudo systemctl status firewalld


# 2. Check current firewall zones and rules

# List active zones
sudo firewall-cmd --get-active-zones

# List current rules in the active zone (usually public)
sudo firewall-cmd --list-all


# 3. Add the port rule for Node Exporter

# Add port 9100 to the public zone
sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent

# Reload firewall to apply changes
sudo firewall-cmd --reload

# Verify the port was added
sudo firewall-cmd --list-all

# 5. Verify Node Exporter is accessible

# Test locally
curl localhost:9100/metrics

# From another machine
curl http://192.168.100.134:9100/metrics

# Thanks for watching!



