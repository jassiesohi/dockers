# Docker Engine Installation Guide - Ubuntu 20.04.6 - Part 1
- VMware Workstation Pro 17.6.1
## Environment Setup

# Prerequisites
- VMware Workstation Pro 17.6.1
- Ubuntu 20.04.6 LTS ISO
- Basic knowledge of Linux and Docker concepts

# System Requirements
- Minimum 2 GB RAM (4 GB recommended)
- 200 GB disk space (think provision 200 GB disk space for Ubuntu 20.04.6 LTS)
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
2. Ubuntu 20.04.6 LTS installation
3. Docker Engine installation and configuration
4. Basic Docker operations verification

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*



# VM Machine Specs:
# 1. 2 CPU
# 2. 4 GB RAM
# 3. 200 GB Disk 
# Machine Name - dockernode01

# Hmmm! Seems Local Mirror Problem!

# Let's solve it!


sudo vi /etc/apt/sources.list
http://mirror.rise.ph/ubuntu/


# Update the apt package index & Network Tools

# Update Package Index

sudo apt update && \    
sudo apt upgrade -y

# Update the Network Tools, jq & curl
sudo apt install net-tools -y && \
sudo apt install jq -y && \
sudo apt install curl -y


# Install Docker Engine on Ubuntu

sudo curl https://get.docker.com/ | bash


dockerd-rootless-setuptool.sh install


docker --version

# Test Docker Installation

docker run hello-world


# List the Namespaces

lsns

ubuntu@dockernode01:~$ lsns
        NS TYPE   NPROCS   PID USER   COMMAND
4026531834 time        8  1401 ubuntu -bash
4026531835 cgroup      8  1401 ubuntu -bash
4026531836 pid         8  1401 ubuntu -bash
4026531837 user        4  1401 ubuntu -bash
4026531838 uts         8  1401 ubuntu -bash
4026531839 ipc         8  1401 ubuntu -bash
4026531840 net         5  1401 ubuntu -bash
4026531841 mnt         4  1401 ubuntu -bash
4026532621 user        4  2991 ubuntu /proc/self/exe --state-dir=/run/user/1000/dockerd-rootless --net=slirp4
4026532656 mnt         3  2991 ubuntu /proc/self/exe --state-dir=/run/user/1000/dockerd-rootless --net=slirp4
4026532657 net         3  2991 ubuntu /proc/self/exe --state-dir=/run/user/1000/dockerd-rootless --net=slirp4
4026532717 mnt         1  3010 ubuntu slirp4netns --mtu 65520 -r 3 --disable-host-loopback --enable-sandbox -

# List the PID Namespaces

lsns -t pid

ubuntu@dockernode01:~$ lsns -t pid
        NS TYPE NPROCS   PID USER   COMMAND
4026531836 pid       8  1401 ubuntu -bash


# Launching Multiple Containers

for i in {1..10}; do docker run -d nginx:latest; done


ubuntu@dockernode01:~$ for i in {1..10}; do docker run -d nginx:latest; done
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
a480a496ba95: Pull complete
f3ace1b8ce45: Pull complete
11d6fdd0e8a7: Pull complete
f1091da6fd5c: Pull complete
40eea07b53d8: Pull complete
6476794e50f4: Pull complete
70850b3ec6b2: Pull complete
Digest: sha256:28402db69fec7c17e179ea87882667f1e054391138f77ffaf0c3eb388efc3ffb
Status: Downloaded newer image for nginx:latest
3019d157caff4ab43cb520c990ac502b77dad9d95cf939c03cf315b6d56f3949
71a660e1b321353794a63d6786d18879155f736a9bd0690186cedc64fd33b287
ce32cd987dd63304753e48bd710ac1bbc923043736949a7fcbf2541127c60f32
6fe93c0d15dafdf6150db6504d615174804ab31fbcfe5c92ce8ca9748e2d4a74
8de85915bce9a092e19a4bc7e787ff6895672deb51642ae6a97a08197cb2e1e0
c1252d14f897a5ad8acdd56d02d90396df5b74481331d424c07500080d1e6196
77c883434f77aa691cf70888b93b00109ae1c86603aaab248cad6b9c4f7a9aba
23a233cc93194056dc37c3b89f4ec02dc07d98e4b2659140de974492bc9e18e4
5c8c7123a8b9ccc2ca973abf988a0816c30a1080f67f226dd7e4b86c370b6bad
58436d6b348647a0ef3f2228230daff65de4f2a1ea74ba307fdd64f7a58e6d4e
ubuntu@dockernode01:~$


# List the running containers

docker ps


ubuntu@dockernode01:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
58436d6b3486   nginx:latest   "/docker-entrypoint.…"   42 seconds ago   Up 42 seconds   80/tcp    elegant_albattani
5c8c7123a8b9   nginx:latest   "/docker-entrypoint.…"   42 seconds ago   Up 42 seconds   80/tcp    youthful_easley
23a233cc9319   nginx:latest   "/docker-entrypoint.…"   43 seconds ago   Up 43 seconds   80/tcp    great_banach
77c883434f77   nginx:latest   "/docker-entrypoint.…"   43 seconds ago   Up 43 seconds   80/tcp    naughty_ganguly
c1252d14f897   nginx:latest   "/docker-entrypoint.…"   43 seconds ago   Up 43 seconds   80/tcp    intelligent_cohen
8de85915bce9   nginx:latest   "/docker-entrypoint.…"   43 seconds ago   Up 43 seconds   80/tcp    objective_diffie
6fe93c0d15da   nginx:latest   "/docker-entrypoint.…"   44 seconds ago   Up 43 seconds   80/tcp    hungry_newton
ce32cd987dd6   nginx:latest   "/docker-entrypoint.…"   44 seconds ago   Up 44 seconds   80/tcp    pensive_chebyshev
71a660e1b321   nginx:latest   "/docker-entrypoint.…"   44 seconds ago   Up 44 seconds   80/tcp    busy_leakey
3019d157caff   nginx:latest   "/docker-entrypoint.…"   44 seconds ago   Up 44 seconds   80/tcp    gracious_wilbur

# Stop all containers

docker stop $(docker ps -aq)

docker ps -a
docker ps -aq

# Remove all containers

docker rm $(docker ps -aq)

ubuntu@dockernode01:~$ ps -a
    PID TTY          TIME CMD
   4768 pts/0    00:00:00 ps
ubuntu@dockernode01:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@dockernode01:~$ docker ps -aq

# Edit the .bashrc file for aliases

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

ubuntu@dockernode01:~$ docker run --rm -d --name my-nginx nginx:latest
00457065f81d57018bcd54fafd8e836b4ee1d5b25d158e917bb3e01af87efcb9
ubuntu@dockernode01:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
00457065f81d   nginx:latest   "/docker-entrypoint.…"   11 seconds ago   Up 10 seconds   80/tcp    my-nginx
ubuntu@dockernode01:~$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
00457065f81d   nginx:latest   "/docker-entrypoint.…"   22 seconds ago   Up 21 seconds   80/tcp    my-nginx
ubuntu@dockernode01:~$ docker stop my-nginx
my-nginx
ubuntu@dockernode01:~$ docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
ubuntu@dockernode01:~$

# Deploy a container with port forwarding

docker run --rm -d --name my-nginx -p 8000:80 nginx:latest

docker ps

curl http://localhost:8000

# Deploy a busybox container

docker run --rm -d --name my-busybox busybox sleep 3600

docker ps

docker exec -it my-busybox ping -c 4 172.18.0.2

# Lets try to chekc if dns resolution is working

docker exec -it my-busybox ping my-nginx

# What happened here? Bad address? This is because the default bridge network does not have dns resolution by default.
# So we need to create a custom bridge network to have dns resolution.
# We will create a custom bridge network in the next part.

# Inspect a container: 
docker inspect my-busybox

# View Docker Container logs
docker logs my-nginx -f

