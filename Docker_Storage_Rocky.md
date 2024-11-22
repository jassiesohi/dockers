# Docker Engine Storage Migration Guide - Rocky Linux 9.4, AlmaLinux 9.4, Oracle Linux 9.4 - Same Steps for all three Linux Distributions
## Environment Setup

# Prerequisites
- VMware Workstation Pro 17.6.1
- Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4 ISO
- Basic knowledge of Linux and Docker concepts

# System Requirements
- Minimum 2 GB RAM (4 GB recommended)
- 200 GB disk space (think provision 200 GB disk space for Rocky Linux 9.4 / AlmaLinux 9.4 / Oracle Linux 9.4)
- 200 GB disk space for Docker storage as additional storage (thin provisioning)
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
3. Additional disk setup for Docker storage
4. Docker Engine storage migration

# Notes
- This is a silent tutorial focused on commands and configurations
- Steps are precise and actionable
- Assumes familiarity with basic Linux commands
- Designed for educational purposes

---
*Disclaimer: This tutorial is for educational purposes only. Always follow best practices and security guidelines in production environments.*


# Verified Method

# Check the lsblk output
lsblk

[admin100@rocky-docker-node0 ~]$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  10.2G  0 rom
nvme0n1     259:0    0   200G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /boot
└─nvme0n1p2 259:2    0   199G  0 part
  ├─rl-root 253:0    0    70G  0 lvm  /
  ├─rl-swap 253:1    0   3.9G  0 lvm  [SWAP]
  └─rl-home 253:2    0 125.1G  0 lvm  /home


# Check current Docker root directory
docker info | grep "Docker Root Dir"

[admin100@rocky-docker-node0 ~]$ docker info | grep "Docker Root Dir"
 Docker Root Dir: /var/lib/docker


# For more detailed storage information
df -h $(docker info | grep "Docker Root Dir" | awk '{print $4}')

[admin100@rocky-docker-node0 ~]$ df -h $(docker info | grep "Docker Root Dir" | awk '{print $4}')
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/rl-root   70G  2.9G   68G   5% /

# Stop Docker Service
sudo systemctl stop docker.service
  
# Stop Docker Socket
sudo systemctl stop docker.socket

# Create a new directory in /home for Docker 
mkdir /home/admin100/docker_storage

# Edit Docker Service File
sudo vi /lib/systemd/system/docker.service 

# Add the following line with the custom directory
#ExecStart=/usr/bin/dockerd -H fd:// -- containerd=/run/containerd/containerd.sock
  
ExecStart=/usr/bin/dockerd --data-root /dockerdata -H fd:// --containerd=/run/containerd/containerd.sock

# Sync the existing Docker data to the new location
sudo rsync -aqxP /var/lib/docker/ /home/admin100/docker_storage
  
# Reload the systemd daemon and start Docker
sudo systemctl daemon-reload && sudo systemctl start docker
  
# Check Docker Status
sudo systemctl status docker --no-pager
  
# Check Docker Process
ps aux | grep -i docker | grep -v grep


# Verify the new location
docker info | grep "Docker Root Dir"

df -h /home/admin100/docker_storage


# If everything works, you can remove the old Docker directory (optional):
sudo rm -rf /var/lib/docker


# Deploy removable containers
docker run --rm -d --name my-nginx nginx:latest

# Check the container
docker ps

# Check the container details
docker ps -a

# Stop the container
docker stop my-nginx

# Check the container details
docker ps -a



# Adding New Storage Drive of 200G for Docker Storage
# Docker engine is using /var/lib/docker as the root directory that used main storage drive (nvme0n1)
# Assuming the disk is getting full, we will use the new 200G disk (nvme0n2) for `/dockerdata0`
# You have already seen that the new disk has been added to the vm in VMware Workstation Pro and set to thin provisioning.
# I'll help you to configure the new 200G disk (nvme0n2) for `/dockerdata0` as new storage directory for Docker Engine. 
# Here are the exact commands based on our system's configuration:

1. Create a new physical partition on nvme0n2:

sudo fdisk /dev/nvme0n2

# In fdisk, enter these commands:
# n (new partition)
# p (primary partition)
# 1 (partition number)
# Enter (default first sector)
# Enter (default last sector - uses all space)
# w (write changes)


# Check the lsblk output

lsblk

[admin100@rocky-docker-node0 ~]$ sudo pvcreate /dev/nvme0n
nvme0n1    nvme0n1p1  nvme0n1p2  nvme0n2    nvme0n2p1
[admin100@rocky-docker-node0 ~]$ sudo pvcreate /dev/nvme0
nvme0      nvme0n1    nvme0n1p1  nvme0n1p2  nvme0n2    nvme0n2p1
[admin100@rocky-docker-node0 ~]$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sr0          11:0    1  10.2G  0 rom
nvme0n1     259:0    0   200G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /boot
└─nvme0n1p2 259:2    0   199G  0 part
  ├─rl-root 253:0    0    70G  0 lvm  /
  ├─rl-swap 253:1    0   3.9G  0 lvm  [SWAP]
  └─rl-home 253:2    0 125.1G  0 lvm  /home
nvme0n2     259:3    0   200G  0 disk
└─nvme0n2p1 259:4    0   200G  0 part


2. Create physical volume:

# Create PV on the new partition
sudo pvcreate /dev/nvme0n2p1

[admin100@rocky-docker-node0 ~]$ sudo pvcreate /dev/nvme0n2p1
  Physical volume "/dev/nvme0n2p1" successfully created.


# Verify PV creation
sudo pvs

[admin100@rocky-docker-node0 ~]$ sudo pvs
  PV             VG Fmt  Attr PSize    PFree
  /dev/nvme0n1p2 rl lvm2 a--  <199.00g       0
  /dev/nvme0n2p1    lvm2 ---  <200.00g <200.00g


3. Create volume group:

# Create new VG named dockervg
sudo vgcreate dockervg /dev/nvme0n2p1

[admin100@rocky-docker-node0 ~]$ sudo vgcreate dockervg /dev/nvme0n2p1
  Volume group "dockervg" successfully created


# Verify VG creation
sudo vgs

[admin100@rocky-docker-node0 ~]$ sudo vgs
  VG       #PV #LV #SN Attr   VSize    VFree
  dockervg   1   0   0 wz--n- <200.00g <200.00g
  rl         1   3   0 wz--n- <199.00g       0


4. Create logical volume:

# Create LV using all space
sudo lvcreate -l 100%FREE -n dockerlv dockervg

[admin100@rocky-docker-node0 ~]$ sudo lvcreate -l 100%FREE -n dockerlv dockervg
  Logical volume "dockerlv" created.


# Verify LV creation
sudo lvs

[admin100@rocky-docker-node0 ~]$ sudo lvs
  LV       VG       Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  dockerlv dockervg -wi-a----- <200.00g
  home     rl       -wi-ao---- <125.08g
  root     rl       -wi-ao----   70.00g
  swap     rl       -wi-ao----   <3.92g



5. Create XFS filesystem:

# Format with XFS (Rocky Linux default)
sudo mkfs.xfs /dev/dockervg/dockerlv

[admin100@rocky-docker-node0 ~]$ sudo mkfs.xfs /dev/dockervg/dockerlv
meta-data=/dev/dockervg/dockerlv isize=512    agcount=4, agsize=13106944 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=52427776, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=25599, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0



6. Create mount point and configure mount:

# Create directory
sudo mkdir -p /dockerdata0

# Add to fstab for persistent mount
echo "/dev/dockervg/dockerlv /dockerdata0 xfs defaults 0 0" | sudo tee -a /etc/fstab

[admin100@rocky-docker-node0 ~]$ echo "/dev/dockervg/dockerlv /dockerdata0 xfs defaults 0 0" | sudo tee -a /etc/fstab
/dev/dockervg/dockerlv /dockerdata0 xfs defaults 0 0

# Mount the filesystem
sudo mount -a

[admin100@rocky-docker-node0 ~]$ sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[admin100@rocky-docker-node0 ~]$ sudo systemctl daemon-reload


# Verify mount
df -h /dockerdata0

[admin100@rocky-docker-node0 ~]$ df -h /dockerdata0
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/dockervg-dockerlv  200G  1.5G  199G   1% /dockerdata0


[admin100@rocky-docker-node0 ~]$ lsblk
NAME                  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sr0                    11:0    1  10.2G  0 rom
nvme0n1               259:0    0   200G  0 disk
├─nvme0n1p1           259:1    0     1G  0 part /boot
└─nvme0n1p2           259:2    0   199G  0 part
  ├─rl-root           253:0    0    70G  0 lvm  /
  ├─rl-swap           253:1    0   3.9G  0 lvm  [SWAP]
  └─rl-home           253:2    0 125.1G  0 lvm  /home
nvme0n2               259:3    0   200G  0 disk
└─nvme0n2p1           259:4    0   200G  0 part
  └─dockervg-dockerlv 253:3    0   200G  0 lvm  /dockerdata0



7. Set permissions and SELinux context:

# Set ownership
sudo chown root:root /dockerdata0

# Set permissions
sudo chmod 755 /dockerdata0

# Set SELinux context for Docker use
sudo semanage fcontext -a -t container_file_t "/dockerdata0(/.*)?"
sudo restorecon -R /dockerdata0


8. Verify setup:

# Check mount point
df -h /dockerdata0

[admin100@rocky-docker-node0 ~]$ sudo chown root:root /dockerdata0
[admin100@rocky-docker-node0 ~]$ sudo chmod 755 /dockerdata0
[admin100@rocky-docker-node0 ~]$ sudo semanage fcontext -a -t container_file_t "/dockerdata0(/.*)?"
[admin100@rocky-docker-node0 ~]$ sudo restorecon -R /dockerdata0
[admin100@rocky-docker-node0 ~]$ df -h /dockerdata0
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/dockervg-dockerlv  200G  1.5G  199G   1% /dockerdata0
[admin100@rocky-docker-node0 ~]$ ls -la /dockerdata0/
total 0
drwxr-xr-x.  2 root root   6 Oct 24 17:32 .
dr-xr-xr-x. 19 root root 254 Oct 24 17:33 ..




# Check LVM configuration
sudo lvs

[admin100@rocky-docker-node0 ~]$ sudo lvs
  LV       VG       Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  dockerlv dockervg -wi-ao---- <200.00g
  home     rl       -wi-ao---- <125.08g
  root     rl       -wi-ao----   70.00g
  swap     rl       -wi-ao----   <3.92g



sudo vgs

[admin100@rocky-docker-node0 ~]$ sudo vgs
  VG       #PV #LV #SN Attr   VSize    VFree
  dockervg   1   1   0 wz--n- <200.00g    0
  rl         1   3   0 wz--n- <199.00g    0

# Check SELinux context
ls -lZd /dockerdata0


[admin100@rocky-docker-node0 ~]$ ls -lZd /dockerdata0/
drwxr-xr-x. 2 root root system_u:object_r:container_file_t:s0 6 Oct 24 17:32 /dockerdata0/


9. Steps to move Docker storage to the new location

# Check current Docker root directory
docker info | grep "Docker Root Dir"

[[admin100@rocky-docker-node0 ~]$ docker info | grep "Docker Root Dir"
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
 Docker Root Dir: /var/lib/docker



# For more detailed storage information
df -h $(docker info | grep "Docker Root Dir" | awk '{print $4}')

[admin100@rocky-docker-node0 ~]$ df -h $(docker info | grep "Docker Root Dir" | awk '{print $4}')
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/rl-root   70G  2.9G   68G   5% /

# Stop Docker Service
sudo systemctl stop docker.service
  
# Stop Docker Socket
sudo systemctl stop docker.socket

# we will use /dockerdata0 as the new Docker root directory
/dockerdata0

# Edit Docker Service File
sudo vi /lib/systemd/system/docker.service 

# Add the following line with the custom directory
#ExecStart=/usr/bin/dockerd -H fd:// -- containerd=/run/containerd/containerd.sock
  
ExecStart=/usr/bin/dockerd --data-root /dockerdata0 -H fd:// --containerd=/run/containerd/containerd.sock

# Sync the existing Docker data to the new location
sudo rsync -aqxP /var/lib/docker/ /dockerdata0
  
# Reload the systemd daemon and start Docker
sudo systemctl daemon-reload && sudo systemctl start docker
  
# Check Docker Status
sudo systemctl status docker --no-pager
  
# Check Docker Process
ps aux | grep -i docker | grep -v grep

[admin100@rocky-docker-node0 ~]$ ps aux | grep -i docker | grep -v grep
root        2419  0.2  2.1 1847736 81512 ?       Ssl  17:46   0:00 /usr/bin/dockerd --data-root /dockerdata0 -H fd:// --containerd=/run/containerd/containerd.sock


# Verify the new location
docker info | grep "Docker Root Dir"

[admin100@rocky-docker-node0 ~]$ docker info | grep "Docker Root Dir"
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
 Docker Root Dir: /dockerdata0


# Verify the new location
df -h /dockerdata0

[admin100@rocky-docker-node0 ~]$ df -h /dockerdata0
Filesystem                     Size  Used Avail Use% Mounted on
/dev/mapper/dockervg-dockerlv  200G  2.1G  198G   2% /dockerdata0


# If everything works, you can remove the old Docker directory (optional):
sudo rm -rf /var/lib/docker


# Deploy removable containers
docker run --rm -d --name my-nginx nginx:latest

# Check the container
docker ps

# Check the container details
docker ps -a

# Stop the container
docker stop my-nginx

# Check the container details
docker ps -a

