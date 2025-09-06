# Week 2: Revisiting the Fundamentals and Exploring New Tools

I decided to set aside this week to reinforce my understanding of key tools and foundational topics. Even though it took a while to go through, I’m glad I went back to the basics. A lot of these tools were things I’d learned but never really applied, so they didn’t stick. 

Thinking back to my time as an SRE, I remember so many instances where leveraging some of these concepts would’ve made troubleshooting so much easier. 

Well... at least I know now! 😅

<img src="https://github.com/jessica-nguyen-it/RHEL9-Homelab-AWS/blob/main/assets/miscellaneous/Funny-businessman-swimming-underwater.jpg?raw=true" alt="Funny businessman underwater" width="500"/>

#### Anyways, to keep things light (and leave room for the more fun stuff), here’s a quick list of what I reviewed:

- Linux security and access control fundamentals: account types, ACLs, privilege escalation, user management, and file ownership/permissions  
- System documentation tools: `--help`, man pages, `apropos`, `mandb`, `info`, and `/usr/share/doc`  
- Hard vs. soft links: how they work and their limitations  
- File search and content manipulation  
- Basic and extended regular expressions  
- Archiving and compression: packing/unpacking files  
- Redirecting input/output  
- File transfers: SCP and SFTP  

## New Territory: Tools I Hadn’t Used Before 😲
While most of this was a refresher, I came across a few topics I hadn’t worked with yet. So, I decided to dig a little deeper into what each one does, how it works, and where it might come in handy. Here’s what I found:

### SUID, SGID, and Sticky Bit permissions  

These extra permission bits affect file execution and directory behavior. I practiced setting and removing them, and saw how they can streamline access or introduce risks if misconfigured. Basically:

- **SUID (4):** Lets a file run with the owner's privileges. Great for controlled elevation (terrible if misconfigured).
- **SGID (2):** Same idea, but for group privileges. On directories, it ensures new files inherit the group.
- **Sticky Bit (1):** On shared directories, it prevents users from deleting each other’s files. 

### File-level backup and sync with `rsync`

Remote synchronization lets you copy files between systems efficiently. I practiced using rsync to mirror directories, both locally and remotely. It’s fast, preserves metadata, and works over SSH. 

These are common ways to use `rsync` for file synchronization:

- Pushing your local directory to a remote system:  
  `rsync -a /local/dir username@ip:/remote/dir`
- Pulling a remote directory to your local machine:  
  `rsync -a username@ip:/remote/dir /local/dir`
- Syncing two directories on the same system:  
  `rsync -a Pictures/ /Backups/Pictures/`

### Full disk imaging with `dd`  

`dd` takes an exact bit-by-bit copy of a disk or partition—this is why it's known as imaging. It’s useful for backups, cloning, and restoration.

To create a disk image, use the command: `sudo dd if=/dev/vda of=diskimage.raw bs=1M status=progress`. This copies the contents of `/dev/vda` into a raw image file.

- **if=** input file (source device)  
- **of=** output file (destination image)  
- **bs=** block size (1M is a common default)  
- **status=progress** shows real-time progress

To restore from a disk image, simply reverse the input and output of the previous command: `sudo dd if=diskimage.raw of=/dev/vda bs=1M status=progress`. This writes the image back to the disk!


<!--- This section is in progress! 🚧 --->

