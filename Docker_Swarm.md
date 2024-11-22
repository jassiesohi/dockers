# Version 0.1 - 10102023

# Docker SWARM 

# Is Docker Swarm automatically enabled?
# No, by default, Docker Swarm is not available

# Types of Nodes in a Docker Swarm
# Manager and worker

# Enable the First Node of a Docker Swarm
docker swarm init

# List Running Services
docker service ls

# Add a Node to a Swarm Cluster
docker swarm join --token <token> --listen-addr <ip:port>

# Can manager nodes run containers?
Yes, manager nodes normally run containers

# Retrieve the Join Token
docker swarm join-token

# List Nodes in a Cluster
docker node ls

# Can you run a 'docker node ls' from a worker node?
No. Docker Swarm commands can only be from manager nodes

# List Services in a Docker Swarm
docker service ls

# List Containers in a Service
docker service ps <service name>

# Remove a Service
docker service rm <service name>

# Remove a Node from a Swarm Cluster
docker node rm <node name>

# Promote a Node from Worker to Manager
docker node promote <node name>

# Change a Node from a Manager to a Worker
docker node demote <node name>

# Map a Host Port to a Container Port -p <host port>: <container port>
# Example:
docker run -p 8080:8080 <image name>