# Week 1 – Spinning Up the Homelab

This week I decided I would dedicate to laying the groundwork: launching the instance, setting up SSH, and installing core tools. I also (unexpectedly) found myself having to troubleshoot an issue, barely out of the box.

It’s okay though, this is exactly the kind of hands-on learning I signed up for, so we’re already off to a great start (●'◡'●)

## Launching the Instance

I started things off by creating an SSH key pair (`rhcsa-key.pem`) and launching a `t3.micro` instance using the [RHEL 9 AMI](https://aws.amazon.com/marketplace/pp/prodview-b5psjqk4f5f3k) from AWS Marketplace.

For storage, I gave it:
- a 20 GiB root volume
- and two 5 GiB EBS volumes for LVM practice *(which, to be honest, I don’t fully understand yet, but I’m getting there)*

I also set up a security group to allow SSH from my IP and made sure the instance had a public IP for remote access.

## Connecting & Prepping the System

Once the instance was running, I needed to restrict permissions on the `.pem` file so SSH would accept it. On Linux, that’s a simple:

```bash
chmod 400 rhcsa-key.pem
ssh -i rhcsa-key.pem ec2-user@<public-ip>
```

But since I was on Windows, I had to use PowerShell and `icacls` to achieve the same effect:

```powershell
icacls rhcsa-key.pem /inheritance:r
icacls rhcsa-key.pem /grant:r "$($env:USERNAME):R"
icacls rhcsa-key.pem /remove "Users" "Administrators" "Everyone"
```

These commands essentially:
- Remove inherited permissions  
- Grant read-only access to my user  
- and Remove access for other groups  

Once that was done, I could connect. And just like that, my homelab was alive! 

## First Hiccup: Installing `sl`

To celebrate, I tried installing my favorite app: the infamous steam locomotive animation (`sl`). But of course, I immediately hit a wall:

```
Error: Unable to find a match: sl
```

Right, `sl` lives in the EPEL repo, which isn’t enabled by default. I registered the instance, enabled the right repos, and manually installed the EPEL release RPM:

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```
Then came the next surprise: the install was killed mid-process. 

Using my super awesome SRE skillz, I ran:

```bash
journalctl -xe | grep -i kill
```

The logs showed that the OOM killer had terminated the `dnf` process during install. That ruled out repo issues or broken packages—it was just my tiny instance running out of steam. This is a `t3.micro` instance with just 1 GiB of RAM, so I guess that makes sense. 

So now, my options were either to change my instance type (which would require my broke self to pay AWS money), or stay within the free tier and practice one of the RHCSA exam objectives I happened to vaguely remember reading about: configuring swap space.

You’ll never guess which one I chose 

```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

After that, `sl` installed perfectly! And yes, the train runs. 🚂

<p align="left">
  <img src="https://github.com/jessica-nguyen-it/RHEL9-Homelab-AWS/blob/main/assets/screenshots/SteamLocomotive.gif?raw=true" alt="Steam Locomotive" width="300"/>
</p>
