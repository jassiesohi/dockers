
# Docker MACVLAN and IPVLAN Setup on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4
## Environment Setup

# Prerequisites
- VMware Workstation Pro 17.6.1
- Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4 VM
- Basic knowledge of Linux and Docker concepts

# System Requirements
- Minimum 2 GB RAM (4 GB recommended)
- 200 GB disk space (think provision 200 GB disk space for Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4)
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
1. Docker MACVLAN and IPVLAN Setup on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*

# Here's a brief comparison of macvlan and ipvlan networks:

## MACVLAN:
- Assigns a unique MAC address to each virtual interface
- Better for environments that require direct Layer 2 communication
- Higher resource overhead due to MAC address management
- Works well when virtual interfaces need to appear as physical NICs

## IPVLAN:
- Shares the parent interface's MAC address for all virtual interfaces
- Operates primarily at Layer 3 (IP level)
- More efficient in terms of resource usage
- Better for large-scale deployments where MAC address management could be a bottleneck
- Useful when you want to prevent Layer 2 communication between containers

# The main difference is that macvlan creates unique MAC addresses for each interface, while ipvlan shares the parent's MAC address.



# Docker MACVLAN and IPVLAN Setup on Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4

# First, check your network interface name
ip addr show

[admin100@rocky-docker-node0 ~]$ ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:67:6b:bb brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.100.134/24 brd 192.168.100.255 scope global noprefixroute ens160
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe67:6bbb/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:a2:9e:37:5c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[admin100@rocky-docker-node0 ~]$


# check the current status of ip_forward
sudo sysctl net.ipv4.ip_forward

[admin100@rocky-docker-node0 ~]$ sudo sysctl net.ipv4.ip_forward
[sudo] password for admin100:
net.ipv4.ip_forward = 1

# if not enabled, enable it with the following commands

# Enable packet forwarding (required for container networking)
# sudo sysctl -w net.ipv4.ip_forward=1

# Make it persistent
# echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-ipforward.conf
# sudo sysctl -p /etc/sysctl.d/99-ipforward.conf

# 1. MACVLAN Setup
# Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=ens160 macvlan_net    # Replace ens160 with your interface name

[admin100@rocky-docker-node0 ~]$ # Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=ens160 macvlan_net    # Replace ens160 with your interface name
e38637dfb60c6c2e3f6b6fafa459dc8abe7280f3aa57afdb50157b18d95b1a66


# List the networks
docker network ls

[admin100@rocky-docker-node0 ~]$ docker network ls
NETWORK ID     NAME          DRIVER    SCOPE
1a437546b46f   bridge        bridge    local
2b4f3e9063a5   host          host      local
e38637dfb60c   macvlan_net   macvlan   local
6ba3d1b149c6   none          null      local


# Stop all containers

docker stop $(docker ps -aq)

docker ps -a
docker ps -aq

# Remove all containers

docker rm $(docker ps -aq)

# Create test containers
docker run --rm -dit --name macvlan_container1 \
  --network macvlan_net \
  --ip=192.168.100.10 \
  alpine sh

docker run --rm -dit --name macvlan_container2 \
  --network macvlan_net \
  --ip=192.168.100.11 \
  alpine sh


# Inspect the created network
docker network inspect macvlan_net

[admin100@rocky-docker-node0 ~]$ docker network inspect macvlan_net
[
    {
        "Name": "macvlan_net",
        "Id": "e38637dfb60c6c2e3f6b6fafa459dc8abe7280f3aa57afdb50157b18d95b1a66",
        "Created": "2024-10-25T13:32:52.517469571+08:00",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
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
        "Containers": {
            "c5a0d873791dda6705f1f5207ca0097ab330d27d65f465decd2427e7230814fe": {
                "Name": "macvlan_container2",
                "EndpointID": "273fef151ef758c00013c2478a09b5123b1aac057ac438b9f00be4a4eeac1fb9",
                "MacAddress": "02:42:c0:a8:64:0b",
                "IPv4Address": "192.168.100.11/24",
                "IPv6Address": ""
            },
            "ec099139bd141cbbdbda287c4c669213ceab89e24ba5cc5612ccbc4aa8d40d1e": {
                "Name": "macvlan_container1",
                "EndpointID": "8098d49b9a35330769bdfbd987b6eda05c623badaf8b0eb15f004ed0ed9be896",
                "MacAddress": "02:42:c0:a8:64:0a",
                "IPv4Address": "192.168.100.10/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "parent": "ens160"
        },
        "Labels": {}
    }
]



# Test connectivity
docker exec -it macvlan_container1 sh
# Inside container1
ping -c 4 192.168.100.11
ip addr show
exit

[admin100@rocky-docker-node0 ~]$ docker exec -it macvlan_container1 sh
/ # ping -c 4 192.168.100.11
PING 192.168.100.11 (192.168.100.11): 56 data bytes
64 bytes from 192.168.100.11: seq=0 ttl=64 time=0.109 ms
64 bytes from 192.168.100.11: seq=1 ttl=64 time=0.147 ms
64 bytes from 192.168.100.11: seq=2 ttl=64 time=0.148 ms
64 bytes from 192.168.100.11: seq=3 ttl=64 time=0.221 ms

--- 192.168.100.11 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.109/0.156/0.221 ms

/ # ping google.com
PING google.com (142.251.221.14): 56 data bytes
64 bytes from 142.251.221.14: seq=0 ttl=56 time=3.283 ms
64 bytes from 142.251.221.14: seq=1 ttl=56 time=4.359 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 3.283/3.821/4.359 ms

/ # ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
15: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:c0:a8:64:0a brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
/ #

# Inspect the macvlan network to check the container's unique MAC address

docker network inspect macvlan_net

[admin100@rocky-docker-node0 ~]$ docker network inspect macvlan_net
[
    {
        "Name": "macvlan_net",
        "Id": "e38637dfb60c6c2e3f6b6fafa459dc8abe7280f3aa57afdb50157b18d95b1a66",
        "Created": "2024-10-25T13:32:52.517469571+08:00",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
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
        "Containers": {
            "c5a0d873791dda6705f1f5207ca0097ab330d27d65f465decd2427e7230814fe": {
                "Name": "macvlan_container2",
                "EndpointID": "273fef151ef758c00013c2478a09b5123b1aac057ac438b9f00be4a4eeac1fb9",
                "MacAddress": "02:42:c0:a8:64:0b",
                "IPv4Address": "192.168.100.11/24",
                "IPv6Address": ""
            },
            "ec099139bd141cbbdbda287c4c669213ceab89e24ba5cc5612ccbc4aa8d40d1e": {
                "Name": "macvlan_container1",
                "EndpointID": "8098d49b9a35330769bdfbd987b6eda05c623badaf8b0eb15f004ed0ed9be896",
                "MacAddress": "02:42:c0:a8:64:0a",
                "IPv4Address": "192.168.100.10/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "parent": "ens160"
        },
        "Labels": {}
    }
]
[admin100@rocky-docker-node0 ~]$

# Lets check that web container nginx needs to have port mapping on macvlan network or not?

# I am creating nginx container on macvlan network without port mapping
docker run --rm -d --name mynginx --network macvlan_net nginx

# inspect the nginx macvlan network
docker network inspect macvlan_net

[admin100@rocky-docker-node0 ~]$ docker network inspect macvlan_net
[
    {
        "Name": "macvlan_net",
        "Id": "e38637dfb60c6c2e3f6b6fafa459dc8abe7280f3aa57afdb50157b18d95b1a66",
        "Created": "2024-10-25T13:32:52.517469571+08:00",
        "Scope": "local",
        "Driver": "macvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
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
        "Containers": {
            "c5a0d873791dda6705f1f5207ca0097ab330d27d65f465decd2427e7230814fe": {
                "Name": "macvlan_container2",
                "EndpointID": "273fef151ef758c00013c2478a09b5123b1aac057ac438b9f00be4a4eeac1fb9",
                "MacAddress": "02:42:c0:a8:64:0b",
                "IPv4Address": "192.168.100.11/24",
                "IPv6Address": ""
            },
            "ec099139bd141cbbdbda287c4c669213ceab89e24ba5cc5612ccbc4aa8d40d1e": {
                "Name": "macvlan_container1",
                "EndpointID": "8098d49b9a35330769bdfbd987b6eda05c623badaf8b0eb15f004ed0ed9be896",
                "MacAddress": "02:42:c0:a8:64:0a",
                "IPv4Address": "192.168.100.10/24",
                "IPv6Address": ""
            },
            "f059a8862d911b188e0b430cbbf2d84cf967d78dee3e15c7e3fb350d83dbe6ef": {
                "Name": "mynginx",
                "EndpointID": "d8d640bee1ed4817bbf743e048ccbe020e679a3a2edeaaa514d2ea2f2fa071d6",
                "MacAddress": "02:42:c0:a8:64:02",
                "IPv4Address": "192.168.100.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "parent": "ens160"
        },
        "Labels": {}
    }
]


# See it worked without port mapping!
# thats because macvlan network is using the same subnet as the host machine & need no port mapping & have its own unique MAC address

# Stop all containers

docker stop $(docker ps -aq)

docker ps -a
docker ps -aq

# Remove all containers

docker rm $(docker ps -aq)

# Remove the macvlan network
docker network rm macvlan_net



# 2. IPVLAN Setup
# Create ipvlan network
docker network create -d ipvlan \
  --subnet=192.168.100.0/24 \
  --gateway=192.168.100.1 \
  -o parent=ens160 \
  -o ipvlan_mode=l2 ipvlan_net

# Inspect the ipvlan network
docker network inspect ipvlan_net

[admin100@rocky-docker-node0 ~]$ docker network inspect ipvlan_net
[
    {
        "Name": "ipvlan_net",
        "Id": "c9c12bfb6a0d2ff979e4c306e6398c9217e2326261a426900d8ca7f56415665f",
        "Created": "2024-10-25T14:01:07.244770035+08:00",
        "Scope": "local",
        "Driver": "ipvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
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
            "ipvlan_mode": "l2",
            "parent": "ens160"
        },
        "Labels": {}
    }
]



# Create test containers
docker run --rm -dit --name ipvlan_container1 \
  --network ipvlan_net \
  --ip=192.168.100.10 \
  alpine sh

docker run --rm -dit --name ipvlan_container2 \
  --network ipvlan_net \
  --ip=192.168.100.11 \
  alpine sh


# Inspect the ipvlan network to check the container's unique MAC address
docker network inspect ipvlan_net

# Test connectivity
docker exec -it ipvlan_container1 sh

# Inside container1
ping -c 4 192.168.100.11
ip addr show
exit


/ # ip add show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
18: eth0@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UNKNOWN
    link/ether 00:0c:29:67:6b:bb brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.10/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever



# Inspect the ipvlan network to check the container's unique MAC address
[admin100@rocky-docker-node0 ~]$ docker network inspect ipvlan_net
[
    {
        "Name": "ipvlan_net",
        "Id": "c9c12bfb6a0d2ff979e4c306e6398c9217e2326261a426900d8ca7f56415665f",
        "Created": "2024-10-25T14:01:07.244770035+08:00",
        "Scope": "local",
        "Driver": "ipvlan",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.100.0/24",
                    "Gateway": "192.168.100.1"
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
        "Containers": {
            "00c34a055e7e6d401fd6222d6d89eb2aa211e19730d31fbe1e677f0dbd7f9909": {
                "Name": "ipvlan_container2",
                "EndpointID": "3aa6acee22e30357c961662a8d8075f59744acf9269b0de51710c1401862f895",
                "MacAddress": "",
                "IPv4Address": "192.168.100.11/24",
                "IPv6Address": ""
            },
            "090378f9110c7e1122deb06a3b83770a2e09c5d418edf958b810c5c2e57d5c09": {
                "Name": "ipvlan_container1",
                "EndpointID": "06402977b560ecf21026abf44a2d1ff3047b003b442815eb12609007aa51c38e",
                "MacAddress": "",
                "IPv4Address": "192.168.100.10/24",
                "IPv6Address": ""
            }
        },
        "Options": {
            "ipvlan_mode": "l2",
            "parent": "ens160"
        },
        "Labels": {}
    }
]



# Cleanup
docker rm -f ipvlan_container1 ipvlan_container2 mynginx

# create a nginx container on ipvlan network without port mapping
docker run --rm -dit --name mynginx --network ipvlan_net nginx

# Inspect the nginx container
docker inspect mynginx

# Remove the nginx container
docker rm -f mynginx

# Remove the ipvlan network
docker network rm ipvlan_net

# Verify networks
docker network ls
docker network inspect macvlan_net
docker network inspect ipvlan_net

# Rocky Linux specific firewall configuration (if needed)
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i macvlan_net -j ACCEPT
sudo firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -o macvlan_net -j ACCEPT
sudo firewall-cmd --reload

# If SELinux is enabled, you might need this
sudo semanage permissive -a container_t

# To check if the networks are properly created
docker network ls --filter driver=macvlan
docker network ls --filter driver=ipvlan


# Thanks for Watching!
