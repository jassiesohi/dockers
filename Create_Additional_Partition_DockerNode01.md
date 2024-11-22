
# Current Disk Layout

ubuntu@dockernode01:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  200G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0  198G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   99G  0 lvm  /
sr0                        11:0    1 1024M  0 rom


# Steps to Create a New Partition in the Volume Group

# Since you have an LVM setup, you can create a new logical volume within the existing volume group (ubuntu-vg). Here’s how to do it step-by-step:

# Step 1: Check Free Space in the Volume Group
sudo vgdisplay ubuntu-vg

ubuntu@dockernode01:~$ sudo vgdisplay ubuntu-vg
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <198.00 GiB
  PE Size               4.00 MiB
  Total PE              50687
  Alloc PE / Size       25343 / <99.00 GiB
  Free  PE / Size       25344 / 99.00 GiB
  VG UUID               tZwaZ5-5qHo-HveG-qpsh-LzYF-84Ep-0eBcuH


# Look for the "Free PE / Size" line to see how much free space you have.

Free  PE / Size       25344 / 99.00 GiB

 Step 2: Create a New Logical Volume

# Assuming you have free space available in your volume group, you can create a new logical volume. For example, to create a logical volume named lv_data with 10G:

sudo lvcreate -L 10G -n lv_data ubuntu-vg

# If you want to use all available free space:

sudo lvcreate -l 100%FREE -n lv_data ubuntu-vg

# Check the file system system of the / root partition


# Use df Command:
# The simplest way to check the filesystem type is by using the df command with the -T option:

df -T /dev/ubuntu-vg/ubuntu-lv

# ubuntu@dockernode01:~$ df -T /dev/ubuntu-vg/ubuntu-lv
# Filesystem                        Type 1K-blocks    Used Available Use% Mounted on
# /dev/mapper/ubuntu--vg-ubuntu--lv ext4 101590008 6924104  89459276   8% /

# Use lsblk Command:
# Another way to check the filesystem type is by using the lsblk command with the -f option:

lsblk -f

# NAME             FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
# sda
├─sda1
├─sda2           ext4        1.0            1491601f-1fc1-42f1-a4e4-dcdcd83035ec      1.7G     5% /boot
└─sda3           LVM2_member LVM2 001       m082TR-lcLc-oni6-VQLN-8Pt5-lduE-LWIrMQ
  └─ubuntu--vg-ubuntu--lv
                 ext4        1.0            116176d5-a957-42b8-ae7d-aa71778a0ee7     85.3G     7% /
sr0

# Use blkid Command:
# Another method is to use the blkid command, which can provide detailed information about block devices, including their filesystem types:

sudo blkid /dev/ubuntu-vg/ubuntu-lv

# ubuntu@dockernode01:~$ sudo blkid /dev/ubuntu-vg/ubuntu-lv
# /dev/ubuntu-vg/ubuntu-lv: UUID="116176d5-a957-42b8-ae7d-aa71778a0ee7" BLOCK_SIZE="4096" TYPE="ext4"




# Step 3: Format the New Logical Volume
sudo mkfs.ext4 /dev/ubuntu-vg/lv_data

# Create a Mount Point
sudo mkdir /mnt/data

# Mount the New Logical Volume
sudo mount /dev/ubuntu-vg/lv_data /mnt/data

# Verify the Setup
df -h /mnt/data


# Make Mount Persistent Across Reboots
# To ensure the mount point persists across reboots, you need to add it to the /etc/fstab file.

sudo blkid /dev/ubuntu-vg/lv_data


ubuntu@dockernode01:~$ sudo mkfs.ext4 /dev/ubuntu-vg/lv_data
[sudo] password for ubuntu:
mke2fs 1.47.0 (5-Feb-2023)
The file /dev/ubuntu-vg/lv_data does not exist and no size was specified.
ubuntu@dockernode01:~$ sudo lvcreate -l 100%FREE -n lv_data ubuntu-vg
  Logical volume "lv_data" created.
ubuntu@dockernode01:~$ sudo mkfs.ext4 /dev/ubuntu-vg/lv_data
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 25952256 4k blocks and 6488064 inodes
Filesystem UUID: 314d5ecb-3e3e-4ef3-8f89-7c8b2470e6f5
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done

ubuntu@dockernode01:~$ sudo mkdir /mnt/data
ubuntu@dockernode01:~$ sudo mount /dev/ubuntu-vg/lv_data /mnt/data
ubuntu@dockernode01:~$ df -h /mnt/data
Filesystem                      Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-lv_data   97G   24K   92G   1% /mnt/data
ubuntu@dockernode01:~$ sudo blkid /dev/ubuntu-vg/lv_data
/dev/ubuntu-vg/lv_data: UUID="314d5ecb-3e3e-4ef3-8f89-7c8b2470e6f5" BLOCK_SIZE="4096" TYPE="ext4"
ubuntu@dockernode01:~$


# Add the following line to the /etc/fstab file:
# /dev/uibuntu-vg/lv_data /mnt/data ext4 defaults 0 2

# Breakdown of the Entry
# /dev/ubuntu-vg/lv_data:
# This specifies the device or logical volume that you want to mount. In this case, it refers to a logical volume named lv_data within the volume group ubuntu-vg. This is where your data will be stored.
# /mnt/data:
# This is the mount point, which is the directory in the filesystem where the logical volume will be attached. Here, lv_data will be accessible at /mnt/data.
# ext4:
# This indicates the filesystem type of the logical volume. In this case, it specifies that the filesystem on lv_data is formatted as ext4, a common and robust filesystem used in Linux.
# defaults:
# This sets a series of default mount options, which typically include:
# rw: Mount the filesystem read-write.
# suid: Allow set-user-identifier or set-group-identifier bits.
# dev: Interpret character or block special devices on the filesystem.
# exec: Allow execution of binaries.
# auto: Can be mounted automatically at boot time.
# nouser: Only root can mount.
# async: All I/O to the filesystem should be done asynchronously.
# 0 (first zero):
# This value indicates whether the filesystem should be dumped (backed up) by the dump command. A value of 0 means it will not be dumped.
# 2 (second number):
# This value specifies the order in which filesystems should be checked at boot time by the fsck command. A value of 2 means that this filesystem will be checked after filesystems with a value of 1. The root filesystem typically has a value of 1, while other filesystems can have a value of 2. A value of 0 means that no check will be performed.


ubuntu@dockernode01:~$ df -H
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              406M  1.6M  405M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  105G  7.2G   92G   8% /
tmpfs                              2.1G     0  2.1G   0% /dev/shm
tmpfs                              5.3M     0  5.3M   0% /run/lock
/dev/sda2                          2.1G   99M  1.9G   6% /boot
tmpfs                              406M   37k  406M   1% /run/user/1000
/dev/mapper/ubuntu--vg-lv_data     105G   25k   99G   1% /mnt/data




# How to change the Docker Root Directory

# Stop Docker Service
sudo systemctl stop docker.service
  
# Stop Docker Socket
sudo systemctl stop docker.socket

# Edit Docker Service File
sudo nano /lib/systemd/system/docker.service 

# Add the following line with the custom directory
  
#ExecStart=/usr/bin/dockerd -H fd:// -- containerd=/run/containerd/containerd.sock
  
ExecStart=/usr/bin/dockerd --data-root /dockerdata -H fd:// --containerd=/run/containerd/containerd.sock

# Sync Docker Data to the New Root Directory
sudo rsync -aqxP /var/lib/docker/ /dockerdata

# Reload Docker Daemon
sudo systemctl daemon-reload && sudo systemctl start docker


# Check Docker Status
sudo systemctl status docker --no-pager
  
# Check Docker Process
ps aux | grep -i docker | grep -v grep





