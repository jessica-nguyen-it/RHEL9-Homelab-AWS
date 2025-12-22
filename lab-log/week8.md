# Week 8 - Mounting on Boot, Disk Compression, and Layered Storage

Officially aced all my final exams! Expect consistent lab logs for the next 2–3 weeks, by then we should be done going through all the topics :]

## Format and Modify Partitions

Before storing files on a partition, you need to create a file system on it. A file system sets up the folders, metadata, and rules the OS needs so it knows where your files begin, end, and how to access them. Without a file system, the partition is just raw space.

#### Create File Systems

Note: the specific flags vary by file system, so always double‑check the mkfs man page if you’re unsure. For example, with `man mkfs.ext4` or `man mkfs.xfs`.

```bash
# Create XFS filesystem
mkfs.xfs /dev/sdXn

# Creating a XFS filesystem with a label
mkfs.xfs -L LabelName /dev/sdXn

# Create XFS with custom inode size
mkfs.xfs -i size=512 /dev/sdXn

```

There are also tools you can use to modify a filesystem’s attributes after creation. For example, typing `xfs` and pressing Tab twice shows all XFS utilities:

```bash
[packetshark@fedora ~]$ xfs
xfs_admin         xfs_estimate       xfs_mkfile         
xfs_bmap          xfs_freeze         xfs_io            
xfs_copy          xfs_fsr            xfs_logprint       
xfs_db            xfs_growfs         xfs_mdrestore      
xfsdump           xfs_info           xfs_metadump      
                  xfs_ncheck         xfs_quota         
                  xfs_repair         xfsrestore  
``` 

Each of these has a man page you can explore, `xfs_admin` is especially useful.

ext4 filesystems have an equivalent tool called `tune2fs`, which you can use like this:

```bash
man tune2fs 
sudo tune2fs -L "SecondFS" /dev/sdb2
```

## Mounting (and mounting on boot!)

This is an important topic, because without it you’d have to manually mount your filesystems every time you boot, which is a mistake I made when I first set up my EC2 instance on AWS. So here’s a quick guide!

#### First, a quick review of the manual way!

In this example, we mount `/dev/vdb1` onto `/mnt/,` a directory used for temporary mounts, and create a file on it. After unmounting the partition, the contents of `/dev/vdb1` will no longer be accessible until it is mounted again.

```bash
# Mount a filesystem manually to the temporary mount directory
sudo mount /dev/vdb1 /mnt/

# Create a test file on the mounted FS
sudo touch /mnt/testfile
ls -l /mnt/

# Unmount (note: no 'n' in umount)
sudo umount /mnt/
```

You can confirm whether the filesystem is mounted or unmounted using `ls /mnt` and `lsblk`.

#### Configuring Permanent Mounts with /etc/fstab

`/etc/fstab` is the file Linux uses to define which filesystems should be mounted automatically at boot, where they should be mounted, their type, mount options, whether they should be backed up (usually no), and the order they should be checked during boot (if at all).

```bash
# Create a permanent mount point
sudo mkdir /mybackups

# Edit fstab
sudo vim /etc/fstab

# fstab fields:
# <device>   <mountpoint>  <fstype>  <options>  <dump>  <fsck>

# Example entries
/dev/vdb1  /mybackups   xfs   defaults   0 2  # xfs
/dev/vdb2  /mybackups   ext4  defaults   0 2  # ext4
/dev/vdb3  none         swap  defaults   0 0  # swap

# Example entry using UUID instead of device name
sudo blkid /dev/vdb1 # Checks UUID of a device
UUID=9ab8cfa5-2813-4b70-ada0-7abd0ad9d289   /mybackups   xfs   defaults   0 2

# Reload systemd units after editing fstab
sudo systemctl daemon-reload
```

Most of the fields are straightforward, but the last two may be new:

	dump — determines whether the filesystem should be backed up by the old dump utility.

        0 = no (almost always)
        1 = yes

    fsck — determines the order in which filesystems are checked at boot.

        0 = do not check
        1 = check first (reserved for /)
        2 = check after root


## Configuring Disk Compression using VDO 

With rumors of SSD prices about to skyrocket alongside RAM, it's important to know how to configure efficient storage management. One tool that can help you do so is VDO!

How does it work?

1. Zero‑Block Filtering

	- First, VDO scans incoming writes and drops any blocks that contain only zeros, since storing them would waste space.

2. Deduplication

	- After filtering zero blocks, VDO compares new blocks to what’s already stored and keeps only one copy of identical data, pointing duplicates to the same block.

3. Compression

	- Finally, After removing zero blocks and duplicates, VDO compresses the remaining unique data to shrink it further and save additional space.

#### In RHEL 9, VDO is fully integrated int LVM. Here's how to set it up:

```bash
# Create PV + VG
sudo pvcreate /dev/vdb
sudo vgcreate vdo_volume /dev/vdb

# Create LV with VDO
sudo lvcreate --type vdo -n vdo_storage -L 100%FREE -V 10G vdo_volume/vdo_pool1

# Create Filesystem (make sure to skip discard)
sudo mkfs.xfs -K /dev/vdo_volume/vdo_storage # OR
sudo mkfs.ext4 -E nodiscard /dev/vdo_volume/vdo_storage

# Mount (in /etc/fstab)
/dev/vdo_volume/vdo_storage /mnt/myvdo xfs defaults 0 0
sudo mount -a

# Verify
df -h /mnt/myvdo
sudo vdostats --human-readable

```


## Managing layered storage with Stratis

Sort of like LVM, Stratis is a layered‑storage manager that simplifies working with pools of block devices. 

It lets you combine disks into a storage pool, create thin‑provisioned XFS filesystems, take snapshots, and expand storage seamlessly. 

#### Installing and Enabling Stratis
```bash
sudo yum install stratisd stratis-cli
sudo systemctl enable --now stratisd.service
```

#### Creating a Pool and Mounting It
```bash
# Create Stratis Pool 
sudo stratis pool create my-pool /dev/vdc /dev/vdd

# Verify Pool
sudo stratis pool list

# Create Filesystem in Pool
sudo stratis fs create my-pool myfs1 
sudo stratis fs 

# Mount Filesystem
sudo mkdir /mnt/mystratis

# /etc/fstab entry:
/dev/stratis/my-pool/myfs1 /mnt/mystratis xfs x-systemd.requires=stratisd.service 0 0

# Mounts what you just added to /etc/fstab
sudo mount -a
```

#### Expanding a Pool (to add a new device)
```bash
sudo stratis pool add-data my-pool /dev/vdd 
sudo stratis pool list
```

#### Creating and Restoring from Snapshots
```bash
# Create Snapshot
sudo stratis fs snapshot my-pool myfs1 myfs1-snapshot 
sudo stratis fs

## Restore from Snapshot

# (1) Delete or lose data 
rm /mnt/mystratis/mydata.txt 

# (2) Rename current filesystem 
sudo stratis fs rename my-pool myfs1 myfs1-old 

# (3) Promote snapshot 
sudo stratis fs rename my-pool myfs1-snapshot myfs1 

# (4) Remount 
sudo umount /mnt/mystratis 
sudo mount /mnt/mystratis 

# (5) Verify 
sudo stratis fs ls /mnt/mystratis

```