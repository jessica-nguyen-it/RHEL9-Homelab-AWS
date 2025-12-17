# Week 7 - Advanced File System Permissions and Disk Quotas

This will be a lighter week as I head into finals season. In this lab log, we'll go over how to create, manage, and diagnose file permissions using ACLs and attributes. Then, we will set up user and group disk quotas for file systems!

Let's get started:

## Access Control Lists (ACLs)

ACLs allow you to define special exceptions for specific users and groups, providing greater granularity and control when managing permissions, especially in edge-case scenarios.

```bash
# Grant user specific permissions 
sudo setfacl --modify user:aaron:rw examplefile 

# View ACLs 
getfacl examplefile 

# Modify ACL mask (limits effective permissions) 
sudo setfacl --modify mask:r-- examplefile 

# Grant group permissions 
sudo setfacl --modify group:wheel:rw examplefile 

# Deny user all access 
sudo setfacl --modify user:aaron:--- examplefile 

# Remove ACL entry for user 
sudo setfacl --remove user:aaron examplefile 

# Remove ACL entry for group 
sudo setfacl --remove group:wheel examplefile 

# Apply ACLs recursively to directory 
sudo setfacl --recursive -m user:aaron:rwx dir1/ 

# Remove ACL recursively 
sudo setfacl --recursive --remove user:aaron dir1/

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
