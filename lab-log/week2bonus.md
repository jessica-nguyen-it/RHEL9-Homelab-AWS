# Bonus Log: I Forgot to Format My EBS Volumes 🐧

Remembering I had configured two extra EBS volumes, I set out to practice disk backup and restoration using the `dd` command. That’s when I remembered I had never formatted the volumes I attached to my EC2 instance. So, here’s a quick walkthrough of how I fixed that!

## 🔍 Spotting the Problem

I ran `df -h` and noticed only the root volume and its boot partitions were mounted. My two additional EBS volumes were nowhere to be seen:

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           359M   84K  359M   1% /dev/shm
tmpfs           144M  2.9M  141M   3% /run
efivarfs        128K  4.4K  119K   4% /sys/firmware/efi/efivars
/dev/nvme0n1p4   19G  3.3G   16G  18% /
/dev/nvme0n1p3  960M  324M  637M  34% /boot
/dev/nvme0n1p2  200M  7.1M  193M   4% /boot/efi
tmpfs            72M     0   72M   0% /run/user/1000
```

Then I ran lsblk to confirm the volumes were there but unmounted:

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0   20G  0 disk
├─nvme0n1p1 259:1    0    1M  0 part
├─nvme0n1p2 259:2    0  200M  0 part /boot/efi
├─nvme0n1p3 259:3    0    1G  0 part /boot
└─nvme0n1p4 259:4    0 18.8G  0 part /
nvme1n1     259:5    0    5G  0 disk
nvme2n1     259:6    0    5G  0 disk
```

Finally to triple-check, I used file -s and confirmed they were raw:

```bash
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ sudo file -s /dev/nvme1n1
/dev/nvme1n1: data
```

## 🛠️ Step-by-Step Fix

The evidence speaks for itself, before I could begin working with `dd` and `rsync` for disk backup and restoration, I needed to ensure the EBS volumes were properly set up. That meant formatting them, creating mount points, and verifying they were ready to store data. Without that foundation, there’s nowhere for the backup or sync operations to go. 

Here’s how I did that!

### 1. Formatting the Volumes

```bash  
sudo mkfs.xfs /dev/nvme1n1  
sudo mkfs.xfs /dev/nvme2n1  
```

Check that the filesystems are now recognized:

```bash  
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ lsblk -f  
NAME        FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
nvme0n1
├─nvme0n1p1
├─nvme0n1p2 vfat   FAT16       7B77-95E7                             192.7M     4% /boot/efi
├─nvme0n1p3 xfs          boot  27bfefa8-69e7-41ce-a184-c487d56cac49  636.8M    34% /boot
└─nvme0n1p4 xfs          root  b838f0f7-0240-46ea-bf53-c811361cbe43   15.5G    17% /
nvme1n1     xfs                8552178a-d912-4c4b-9c3d-c07da8120eb3    4.9G     1% /mnt/vol1
nvme2n1     xfs                9e3da813-ccfa-4da6-939a-0ff850af9e67    4.9G     1% /mnt/vol2  
```

### 2. Creating Mount Points

```bash  
sudo mkdir /mnt/vol1  
sudo mkdir /mnt/vol2  
```

### 3. Mounting the Volumes

```bash  
sudo mount /dev/nvme1n1 /mnt/vol1  
sudo mount /dev/nvme2n1 /mnt/vol2  
```

### 4. Verifying my Configurations

```bash  
[ec2-user@ip-xxx-xxx-xxx-xxx ~]$ df -h  
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        4.0M     0  4.0M   0% /dev
tmpfs           359M   84K  359M   1% /dev/shm
tmpfs           144M  2.9M  141M   3% /run
efivarfs        128K  4.4K  119K   4% /sys/firmware/efi/efivars
/dev/nvme0n1p4   19G  3.3G   16G  18% /
/dev/nvme0n1p3  960M  324M  637M  34% /boot
/dev/nvme0n1p2  200M  7.1M  193M   4% /boot/efi
tmpfs            72M     0   72M   0% /run/user/1000
/dev/nvme1n1    5.0G   68M  4.9G   2% /mnt/vol1
/dev/nvme2n1    5.0G   68M  4.9G   2% /mnt/vol2
```

Boom. Mounted and ready to go!
