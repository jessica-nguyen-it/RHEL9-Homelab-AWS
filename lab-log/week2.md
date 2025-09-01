# Week 2: Revisiting the Fundamentals and Exploring New Tools

I decided to set aside this week to reinforce my understanding of key tools and foundational topics. Even though it took a while to go through, I’m glad I went back to the basics. A lot of these tools were things I’d learned but never really applied, so they didn’t stick. 

Thinking back to my SRE internship, I realize how much time I could’ve saved if I’d used the built-in documentation tools instead of constantly switching tabs to look through the company’s knowledge base. I also could’ve spared my eyes by using the `diff` command to compare healthy and unhealthy configuration files—among many other little things. 

Well... now I know 😅

<img src="https://github.com/jessica-nguyen-it/RHEL9-Homelab-AWS/blob/main/assets/miscellaneous/Funny-businessman-swimming-underwater.jpg?raw=true" alt="Funny businessman underwater" width="500"/>

To keep things light (and leave room for the more fun stuff), here’s a quick list of what I reviewed:

- Linux security and access control fundamentals: account types, ACLs, privilege escalation, user management, and file ownership/permissions  
- System documentation tools: `--help`, man pages, `apropos`, `mandb`, `info`, and `/usr/share/doc`  
- Hard vs. soft links: how they work and their limitations  
- File search and content manipulation  
- Basic and extended regular expressions  
- Archiving and compression: packing/unpacking files  
- Redirecting input/output  
- File transfers: SCP and SFTP  

## New Territory: Tools I Hadn’t Used Before  
While most of this was a refresher, I came across a few topics I hadn’t worked with before:

- SUID, SGID, and Sticky Bit permissions  
- File-level backup and sync with `rsync`  
- Full disk imaging with `dd`  

So, I decided to dig a little deeper into what each one does, how it works, and where it might come in handy. Here’s what I found:

### SUID, SGID, and Sticky Bit  
This was one of those topics I’d heard mentioned but never really explored. I learned how these special permission bits affect file execution and directory behavior—especially in multi-user environments. I practiced setting and removing them, and saw how they can streamline access or introduce risks if misconfigured.

### rsync  
I’d seen `rsync` used in scripts before, but hadn’t tried it myself until now. Once I got hands-on, I understood why it’s a go-to for file-level backups and sync. I tested syncing directories, preserving permissions, compressing data during transfer, and excluding files. It’s fast, flexible, and surprisingly intuitive.

### dd  
`dd` felt intimidating at first, but once I broke it down, it started to make sense. I used it to create and restore disk images, and learned how powerful it can be for cloning drives or recovering from system failures. Definitely a tool I’ll keep in my back pocket for more advanced backup workflows.
