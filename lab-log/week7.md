# Week 7 - Advanced File System Permissions and Disk Quotas

This will be a lighter week as I head into finals season. In this lab log, we'll go over how to create, manage, and diagnose file permissions using ACLs and attributes. Then, we will set up user and group disk quotas for file systems!

Let's get started:

## Access Control Lists (ACLs)

ACLs allow you to define special exceptions for specific users and groups, providing greater granularity and control when managing permissions, especially in edge-case scenarios.

```bash
# Grant user specific permissions 
sudo setfacl --modify user:packetshark:rw examplefile 

# View ACLs 
getfacl examplefile 

# Modify ACL mask (limits effective permissions) 
sudo setfacl --modify mask:r-- examplefile 

# Grant group permissions 
sudo setfacl --modify group:hiddenleaf:rw examplefile 

# Deny user all access 
sudo setfacl --modify user:packetshark:--- examplefile 

# Remove ACL entry for user 
sudo setfacl --remove user:packetshark examplefile 

# Remove ACL entry for group 
sudo setfacl --remove group:hiddenleaf examplefile 

# Apply ACLs recursively to directory 
sudo setfacl --recursive -m user:packetshark:rwx dir1/ 

# Remove ACL recursively 
sudo setfacl --recursive --remove user:packetshark dir1/

```

## File Attributes

File attributes are useful for enforcing special behaviors on files beyond standard permissions. They let you lock files as immutable (cannot be changed or deleted), make them append‑only, and many more (but we will only go over those two today).

For more options, just paste `man chattr` into your terminal :]

#### Append-Only Attribute (+a)
```bash 
# Enable append-only (can only add data, not overwrite/delete)
sudo chattr +a filename

# Disable append-only
sudo chattr -a filename

# Example: Protect log files so they can only grow
sudo chattr +a /var/log/syslog
```


#### Immutable Attribute (+i)

```bash
# Enable immutable (file cannot be modified, deleted, renamed, or linked)
sudo chattr +i filename

# Disable immutable
sudo chattr -i filename

# Example: Lock critical config file
sudo chattr +i /etc/passwd
```

## Disk Quotas

Disk quotas are limits set on how much disk space or how many files a user or group can use on a filesystem. They help prevent any single user from consuming too much storage, ensuring fair resource distribution and protecting system stability.

#### 1. Installation and Initial Setup

```bash
# 1. Install quota management utilities (if not already installed)
sudo dnf install quota

# 2. Enable quotas in /etc/fstab 
sudo vi /etc/fstab
/dev/vdb1 /mybackups xfs defaults,usrquota,grquota 0 2 sudo vim /etc/fstab 

# 3. Apply changes by rebooting the system 
sudo systemctl reboot 

# 4. For ext4 filesystems, create quota files (aquota.user and aquota.group) 
sudo quotacheck --create-files --user --group /dev/vdb2

```


#### 2. Creating a test environment to demonstrate quota management

```bash
# 1. Create a user directory (XFS example) and generate a test file
sudo mkdir /mybackups/packetshark/
sudo chown packetshark:packetshark /mybackups/packetshark
fallocate --length 100M /mybackups/packetshark/100Mfile

# 2. Adjust user quotas with edquota (set soft/hard limits, e.g. 150M soft, 200M hard)
sudo edquota --user packetshark

# 3. Test quota enforcement by exceeding the soft limit
fallocate --length 60M /mybackups/packetshark/60Mfile
sudo quota --user packetshark

# 4. Attempt to exceed the hard limit (will fail with "Disk quota exceeded")
fallocate --length 40M /mybackups/packetshark/40Mfile
```

#### 3. Inodes, Grace Periods, and Groups

```bash
# 1. Manage inode quotas (limit number of files/directories a user can create)
sudo edquota --user packetshark
# Editor will show inode usage and allow setting soft/hard limits

# 2. Adjust grace periods (time before soft limits are enforced)
sudo edquota -T packetshark

# 3. Manage group quotas (apply quotas to groups instead of individual users)
sudo edquota -g groupname
sudo quota -g groupname
```
