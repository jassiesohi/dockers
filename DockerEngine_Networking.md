
# Docker Engine Networking - Ubuntu 22.04.5 LTS

## Environment Setup
# Prerequisites

- VMware Workstation Pro 17.6.1
- Ubuntu 22.04.5 LTS ISO
- Basic knowledge of Linux and Docker concepts

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
1. Default Bridge Network behavior
2. Custom Bridge Network behavior
3. None Network behavior
4. Host Network behavior    

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*


# Docker Default Bridge Network behavior

# Key Points:
# Default Bridge Network:
# When containers are started without specifying a network, they are placed on the default bridge network (docker0). This network is designed primarily for basic connectivity between containers and the host.
# Default Bridge Network does not allow dns resolution by default.


# List Docker Networks

docker network ls

ubuntu@ubuntu-docker-node0:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
457a6d1a5ce9   bridge    bridge    local
79b83f498da0   host      host      local
c90ac5169135   none      null      local

# Inspect the Bridge Network
docker network inspect bridge

ubuntu@ubuntu-docker-node0:~$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "457a6d1a5ce9ea1d36a9e7f77d1f51b732829f66df74387f86ae8242def427ce",
        "Created": "2024-10-24T05:08:54.581374571Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]

# Check the ip address of the docker0 bridge network

ubuntu@ubuntu-docker-node0:~$ ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:c4:d8:58 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.130/24 brd 192.168.100.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fec4:d858/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:9b:38:6d:cb brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever


# Types of Docker Networks in this tutorial

# Bridge Network
# None Network
# Host Network




# Lets create a User-Defined Bridge Network to have better isolation & enable dns resolution between containers

# Create a User-Defined Bridge Network

docker network create --driver bridge my-custom-bridge-network

docker network ls

ubuntu@ubuntu-docker-node0:~$ docker network ls
NETWORK ID     NAME                       DRIVER    SCOPE
457a6d1a5ce9   bridge                     bridge    local
79b83f498da0   host                       host      local
45f6dda57449   my-custom-bridge-network   bridge    local
c90ac5169135   none                       null      local

# Inspect the custom bridge network
docker network inspect my-custom-bridge-network

# Delete the custom bridge network
docker network rm my-custom-bridge-network

ubuntu@dockernode1:~$ docker network rm my-custom-bridge-network
my-custom-bridge-network
ubuntu@dockernode1:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
d770dfa0571d   bridge    bridge    local
1b3b09d6d817   host      host      local
fd5ea28b8d22   none      null      local


# Create two busybox containers with custom bridge network & check connectivity between them with ping & dns resolution

docker run --rm -d --name my-busybox --network my-custom-bridge-network busybox sleep 3600
docker run --rm -d --name my-busybox1 --network my-custom-bridge-network busybox sleep 3600

docker ps

# Inspect the containers & the custom bridge network
# the ip address of the containers should be in the 172.18.0.0/16 subnet & the gateway should be 172.18.0.1
docker inspect my-busybox
docker inspect my-busybox1
# the ip address of the containers are 172.18.0.2 & 172.18.0.3

# Inspect the custom bridge network
docker network inspect my-custom-bridge-network


# Busybox Container execution

docker exec -it my-busybox ping -c 4 my-busybox1

ubuntu@ubuntu-docker-node0:~$ docker exec -it my-busybox ping -c 4 172.18.0.3
PING 172.18.0.3 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.100 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.152 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.156 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.168 ms

--- 172.18.0.3 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.100/0.144/0.168 ms


# ping the busybox1 container from the busybox container to check DNS resolution
docker exec -it my-busybox ping -c 4 my-busybox1

ubuntu@ubuntu-docker-node0:~$ docker exec -it my-busybox ping -c 4 my-busybox1
PING my-busybox1 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.039 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.057 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.151 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.061 ms

--- my-busybox1 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.039/0.077/0.151 ms

# you can see that dns resolution is working between the containers as this is a user-defined bridge network

# Lets create a nginx container on the custom bridge network & check the connectivity from mapped port 8080 on the host to the nginx container

docker run --rm -d --name my-nginx --network my-custom-bridge-network -p 8080:80 nginx

# Inspect the user-defined bridge network to see that the nginx container is added to the network
docker network inspect my-custom-bridge-network

# check the connectivity from the host to the nginx container
curl http://192.168.100.130:8080

# you can see the nginx welcome page


# Now lets create a container with none network
# None Network
# None network does not have any network connectivity
# its only used for isolation & testing

# stop all running containers
docker stop $(docker ps -q)

ubuntu@ubuntu-docker-node0:~$ docker stop $(docker ps -q)
e36511b572ab
f6f42a0590c4
bf69790cb510

# create a container with none network

docker network ls

ubuntu@ubuntu-docker-node0:~$ docker network ls
NETWORK ID     NAME                       DRIVER    SCOPE
457a6d1a5ce9   bridge                     bridge    local
79b83f498da0   host                       host      local
45f6dda57449   my-custom-bridge-network   bridge    local
c90ac5169135   none                       null      local

# Inspect the none network
docker network inspect none

# create a container with none network
docker run --rm -d --name my-busybox --network none busybox sleep 300

docker network inspect none

# you can see that the container is not connected to any network & have no ip address

# check the ip address of the container
docker exec my-busybox ip addr

# only lo is available which is the loopback interface

ubuntu@ubuntu-docker-node0:~$ docker exec my-busybox ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever



# Lastly Lets create a container with host network
# Host network is used to connect containers directly to the host's network stack
# Containers on host network can directly access the host's network interfaces & services
# Containers on host network can be accessed directly by other hosts on the same network
# its useful to have a container such as prometheus node exporter which needs to scrape the host's metrics

# lets create a prometheus node exporter container on the host network

# stop all running containers
docker stop $(docker ps -q)

# list the networks
docker network ls

# create prometheus node exporter container on the host network
docker run --rm -d --name my-prometheus-node-exporter --network host prom/node-exporter

# create a nginx container on the host network
docker run --rm -d --name my-nginx --network host nginx

docker ps

# check the connectivity from the host to the prometheus node exporter container
curl http://localhost:9100/metrics
curl http://192.168.100.134:9100/metrics


# Set up the firewall rules in Rocky Linux 9.4 using `firewalld` to allow access to the Node Exporter running on port 9100.

# Here are the steps:

# 1. First, verify firewalld is running:

sudo systemctl status firewalld


# 2. Check current firewall zones and rules:

# List active zones
sudo firewall-cmd --get-active-zones

# List current rules in the active zone (usually public)
sudo firewall-cmd --list-all


# 3. Add the port rule for Node Exporter:

# Add port 9100 to the public zone
sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent

# Reload firewall to apply changes
sudo firewall-cmd --reload

# Verify the port was added
sudo firewall-cmd --list-all

# 5. Verify Node Exporter is accessible:

# Test locally
curl localhost:9100/metrics

# From another machine
curl http://192.168.100.134:9100/metrics

# Delete the custom bridge network
docker network rm my-custom-bridge-network


ubuntu@dockernode1:~$ docker network rm my-custom-bridge-network
my-custom-bridge-network
ubuntu@dockernode1:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
d770dfa0571d   bridge    bridge    local
1b3b09d6d817   host      host      local
fd5ea28b8d22   none      null      local



























# NOT REQUIRED IN YOU TUBE VIDEO

##  4. Optional but recommended - Restrict access to specific IP addresses or networks:

# If you want to allow access only from specific IP
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="9100" accept' --permanent

# Reload firewall
sudo firewall-cmd --reload


# 6. If still having issues, check SELinux status:

# Check SELinux status
getenforce

# If it's enforcing, you might need to add SELinux rule
sudo semanage port -a -t http_port_t -p tcp 9100

# Or temporarily set to permissive for testing
sudo setenforce 0


# 7. Double-check the container is running with host networking:

docker ps | grep node-exporter


# Common troubleshooting:

1. If changes don't take effect:

# Remove the rule and add again
sudo firewall-cmd --zone=public --remove-port=9100/tcp --permanent
sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent
sudo firewall-cmd --reload


# 2. To check if the port is actually listening:

ss -tulpn | grep 9100


# 3. To verify no conflicts with SELinux:

# Check SELinux alerts
sudo ausearch -m AVC -ts recent


# Let me know if you need any clarification or run into specific errors!

#########################################################

# For Ubuntu Editions - # NOT REQUIRED IN YOU TUBE VIDEO

# check iptables on the host to see that the node exporter is scraping the metrics from the host
sudo iptables -L -v -n -x -t raw

# allow the ubuntu firewall to allow the node exporter to scrape the metrics from the host
sudo ufw allow 9100/tcp

# restart ubuntu firewall
sudo ufw restart

# allow the node exporter to scrape the metrics from the host
curl -X POST http://192.168.100.130:9100/metrics

# you can see the metrics of the host



# Delete the custom bridge network
docker network rm my-custom-bridge-network


ubuntu@dockernode1:~$ docker network rm my-custom-bridge-network
my-custom-bridge-network
ubuntu@dockernode1:~$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
d770dfa0571d   bridge    bridge    local
1b3b09d6d817   host      host      local
fd5ea28b8d22   none      null      local


