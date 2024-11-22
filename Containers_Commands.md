


# Here are the commands to run Ubuntu container in the background (detached mode):

# Create and start container in detached mode:

docker run -it -d --name my_ubuntu ubuntu

# The `-d` flag runs the container in the background

# Alternative method with a simple process to keep it alive:

docker run -it -d --name my_ubuntu ubuntu tail -f /dev/null


# Or using `sleep infinity`:

docker run -it -d --name my_ubuntu ubuntu sleep infinity


# To verify it's running:

docker ps


# To access the container while it's running in background:

docker exec -it my_ubuntu 


# Additional useful commands:

# To stop the container:

docker stop my_ubuntu


# To restart:

docker restart my_ubuntu


# To remove container (must be stopped first):

docker stop my_ubuntu
docker rm my_ubuntu


# To automatically remove container when stopped:

docker run -it -d --rm --name my_ubuntu ubuntu


# Best Practices:
# 1. Using `tail -f /dev/null` or `sleep infinity` is better than just running with `-d` as it ensures the container stays alive
# 2. If you need the container to automatically restart after system reboot, add `--restart unless-stopped`:

docker run -it -d --restart unless-stopped --name my_ubuntu ubuntu sleep infinity


# Note: When you use `docker exec` to access the container, you can exit  without stopping the container. The container will continue running in the background.


Here are the commands to install IP utilities in an Ubuntu container:

1. First update package list:

apt-get update


2. Install net-tools (for ifconfig, netstat, etc.):

apt-get install -y net-tools


3. Install iproute2 (for ip command):

apt-get install -y iproute2


4. Install additional networking utilities:

apt-get install -y iputils-ping     # for ping
apt-get install -y traceroute       # for traceroute
apt-get install -y dnsutils         # for dig, nslookup
apt-get install -y curl             # for curl
apt-get install -y wget             # for wget
apt-get install -y netcat          # for nc command


5. All-in-one command to install everything:

apt-get update && apt-get install -y \
    net-tools \
    iproute2 \
    iputils-ping \
    traceroute \
    dnsutils \
    curl \
    wget \
    netcat


Common commands after installation:

ip a                   # Show IP addresses (new style)
ifconfig              # Show IP addresses (old style)
ip route              # Show routing table (new style)
route                 # Show routing table (old style)
netstat -tulpn        # Show listening ports
ping google.com       # Test connectivity
traceroute google.com # Trace route to destination
dig google.com        # DNS lookup
nslookup google.com   # Another DNS lookup tool
curl ifconfig.me      # Check external IP


Note: 
- You might need to run these commands with `sudo` if not logged in as root
- Using `-y` flag with apt-get automatically answers "yes" to prompts
- If you get "command not found" errors, run `apt-get update` first
