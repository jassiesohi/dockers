Here are the commands for working with BusyBox containers:

1. Basic BusyBox container creation:

# Pull and run basic BusyBox
docker run -it --name my_busybox busybox

# Run in detached mode
docker run -d --name my_busybox busybox sleep infinity

# Run with specific version
docker run -it --name my_busybox busybox:latest


2. Access container while running in background:

docker exec -it my_busybox sh


3. Basic networking commands available in BusyBox:

# Check IP configuration
ifconfig
ip addr

# Test connectivity
ping google.com

# DNS lookup
nslookup google.com

# Network trace
traceroute google.com

# Show network statistics
netstat

# Download files
wget http://example.com/file


4. File system commands:

# Create directory
mkdir /data

# Create file
touch file.txt

# List files
ls -la

# File manipulation
cp source.txt dest.txt
mv old.txt new.txt
rm file.txt

# Search files
find / -name "filename"


5. System information:

# Show processes
ps aux

# Show memory usage
free

# Show disk usage
df -h

# Show system information
uname -a


6. Container management:

# Stop container
docker stop my_busybox

# Start container
docker start my_busybox

# Restart container
docker restart my_busybox

# Remove container
docker rm -f my_busybox


7. Running with volume mounting:

# Mount host directory
docker run -it --name my_busybox \
    -v $(pwd)/data:/data \
    busybox


8. Network configuration:

# Create custom network
docker network create busybox_net

# Run with custom network
docker run -it --name my_busybox \
    --network busybox_net \
    busybox

# Run with host network
docker run -it --name my_busybox \
    --network host \
    busybox


9. Running with environment variables:

docker run -it --name my_busybox \
    -e MY_VAR="value" \
    busybox


10. Common utilities in BusyBox:

# Text processing
cat file.txt
grep "pattern" file.txt
sed 's/old/new/' file.txt
awk '{print $1}' file.txt

# Compression
tar -cvf archive.tar files/
tar -xvf archive.tar
gzip file.txt
gunzip file.txt.gz

# Process management
top
kill PID
killall process_name


11. Running with specific user:

docker run -it --name my_busybox \
    --user 1000:1000 \
    busybox


12. Persistence and data management:

# Create volume
docker volume create busybox_data

# Run with volume
docker run -it --name my_busybox \
    -v busybox_data:/data \
    busybox


Useful tips:
- BusyBox is very lightweight and includes stripped-down versions of common Unix utilities
- Great for debugging and testing
- Use `ash` or `sh` as shell (no bash by default)
- Limited functionality compared to full Linux distributions
- Perfect for minimal containers and microservices

Remember:
- BusyBox containers are minimal by design
- Some common Linux commands might be missing or have limited functionality
- Great for small footprint containers
- Useful for troubleshooting and network testing
- Consider security implications when exposing services


# BusyBox typically comes with a minimal set of utilities. Let me break down what's available by default and how to install additional tools.

Default Built-in Tools in BusyBox:

# Check available commands
busybox --list

# Common built-in tools:
ash              # Shell
cat              # View file contents
cp               # Copy files
dd               # Convert and copy files
df               # Disk usage
du               # Directory space usage
echo             # Print text
grep             # Search text
ls               # List directory
mkdir            # Create directory
mv               # Move files
ping             # Network ping
ps               # Process status
rm               # Remove files
sed              # Stream editor
tar              # Archive utility
traceroute       # Network route tracing
wget             # Download files
vi               # Text editor


Notably Missing Tools (including curl):
- curl (not included by default)
- ssh
- telnet
- git
- dig
- nmap

To install additional tools in BusyBox, you need to use its package manager `apk`. Here's how:


# Update package index
apk update

# Install curl
apk add curl

# Install other common tools
apk add --no-cache \
    curl \
    git \
    bind-tools \    # for dig, nslookup
    openssh \
    tcpdump \
    nmap \
    vim

# Check installed packages
apk list --installed


However, if you're using the minimal BusyBox image, `apk` might not be available. In this case, you have two options:

1. Use BusyBox Alpine variant:

# Pull and run Alpine-based BusyBox
docker run -it --name my_busybox alpine

# Now you can use apk to install packages
apk update && apk add curl


2. Create a custom Dockerfile:
dockerfile
FROM alpine:latest
RUN apk update && apk add --no-cache \
    curl \
    wget \
    bind-tools \
    openssh
CMD ["/bin/sh"]


Build and run:

docker build -t custom-busybox .
docker run -it --name my_busybox custom-busybox


Testing installed tools:

# Check curl version
curl --version

# Test curl
curl -I https://www.google.com

# Check wget (built-in)
wget --version

# Test wget
wget -q -O - http://example.com


Common Package Combinations:

1. Network Troubleshooting:

apk add --no-cache \
    curl \
    bind-tools \
    tcpdump \
    nmap \
    netcat-openbsd


2. Development Tools:

apk add --no-cache \
    git \
    vim \
    make \
    gcc


3. Text Processing:

apk add --no-cache \
    grep \
    sed \
    awk \
    jq


Remember:
- BusyBox is designed to be minimal
- Installing additional packages increases container size
- Consider if you really need additional tools
- Use Alpine-based images if you need package management
- Some tools might have limited functionality compared to full versions

Best Practices:
1. Only install necessary tools
2. Use multi-stage builds if building custom images
3. Consider security implications of installed tools
4. Document any additional tools installed
5. Keep container images as small as possible




# draft work in progress

# Check the connectivity from the busybox container to the nginx container
docker exec -it my-busybox curl my_nginx


docker run --rm -d --name my_nginx --network my-custom-bridge-network -p 8080:80 nginx
docker run --rm -it -d --name my_ubuntu --network my-custom-bridge-network ubuntu sleep infinity

# Busybox Container execution

docker exec -it my-busybox ping -c 4 my-busybox1
docker exec -it my-busybox curl my_nginx


# Install ping utility in ubuntu container
docker exec -it my-ubuntu apt update && apt install -y iputils-ping



ubuntu@dockernode1:~$ docker run --rm -d --name my_nginx --network my-custom-bridge-network -p 808  0:80 nginx
5a12a6fe159eff651796f8c30ba3b25a40dcd940c56013689a37dc748eebc8b7
ubuntu@dockernode1:~$ docker exec -it my_ubuntu curl my_nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>






# Inspect the custom bridge network
docker network inspect my-custom-bridge-network


ubuntu@dockernode1:~$ docker network inspect my-custom-bridge-network
[
    {
        "Name": "my-custom-bridge-network",
        "Id": "9d254c706843d46f4481afdaf5c8d8bacbab5848458a2ba87db264d21747a87b",
        "Created": "2024-10-23T00:05:28.16071049Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
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
            "5a12a6fe159eff651796f8c30ba3b25a40dcd940c56013689a37dc748eebc8b7": {
                "Name": "my_nginx",
                "EndpointID": "671962094daefdc0bd269c97cc8e194757a737e2d60bbecd0559be7b2a134763",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "5b738f6148b1843d6fdf74b00ea7dce358181cf7acf651021c8d44891f2b81ce": {
                "Name": "my-busybox1",
                "EndpointID": "b5686129d60c6953d275883df1f1b5b6c926ee2e941a9a0ba76b48f0e67e8605",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.5/16",
                "IPv6Address": ""
            },
            "805cd1536988e933c7455d8074f879af6db98e3d01edfc232006c1648e14b133": {
                "Name": "my_ubuntu",
                "EndpointID": "375c1034fcb729191ec072056b5a63a88afd69dba15f685d1bae543b0f3a1994",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "e81b2fee260c899dc01342129ca76a62d3c7a7daf12ea90c77e3ddb7a830a728": {
                "Name": "my-busybox",
                "EndpointID": "235fe0ad3b9597645679f40b140f3605d356e7af52ae8a6b4d0539fd05add7dc",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]


ubuntu@dockernode1:~$ docker exec -it my-busybox ping -c 4 172.18.0.3
PING 172.18.0.3 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.075 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.156 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.156 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.073 ms

--- 172.18.0.3 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.073/0.115/0.156 ms
ubuntu@dockernode1:~$ docker exec -it my-busybox ping -c 4 my-busybox1
PING my-busybox1 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.045 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.151 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.151 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.226 ms

--- my-busybox1 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.045/0.143/0.226 ms

# Ping the busybox1 container from the busybox container

docker exec -it my-busybox ping -c 4 172.18.0.3


ubuntu@dockernode1:~$ docker exec -it my-busybox ping -c 4 172.18.0.3
PING 172.18.0.3 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.156 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.083 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.155 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.161 ms


# Ping the busybox1 container from the busybox container to check DNS resolution
docker exec -it my-busybox ping -c 4 my-busybox1



ubuntu@dockernode1:~$ docker exec -it my-busybox ping -c 4 my-busybox1
PING my-busybox1 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.124 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.164 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.161 ms
64 bytes from 172.18.0.3: seq=3 ttl=64 time=0.182 ms



ubuntu@dockernode1:~$ docker run --rm -d --name my_nginx --network my-custom-bridge-network -p 808  0:80 nginx
5a12a6fe159eff651796f8c30ba3b25a40dcd940c56013689a37dc748eebc8b7
ubuntu@dockernode1:~$ docker exec -it my_ubuntu curl my_nginx
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>