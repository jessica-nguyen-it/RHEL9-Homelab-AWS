# RHEL 9 Homelab on AWS EC2 🐧🧠 

## Why? (a quick blurb before the commands hit the fan...)

I spent my summer as a SRE intern and quickly realized how much I enjoyed working in the terminal. The job centered around maintaining service uptime, and nearly all of those services ran on Linux. Most of my time went into debugging logs, inspecting processes, troubleshooting networking, and tuning performance at the OS level.

Now that summer’s over, I don’t want those skills gathering dust. I’ve got momentum, and I’m not slowing down anytime soon. So, I spun up a RHEL 9 instance on AWS EC2 and built a homelab that mirrors the kind of environments I hope to work in professionally one day.

## The End Goal

With enough trial, error, and terminal time, I plan to translate this hands-on practice into a RHCSA certification and real-world readiness. 

Now, with all that context out of the way, let’s get to what you're actually here for 😹

## 🛠️ Environment
- **Platform**: AWS EC2 (t2.micro)
- **OS**: Red Hat Enterprise Linux 9

## 📚 Series Breakdown

This homelab is documented as a multi-part series, with each entry focused on a specific RHCSA domain. Each part includes screenshots, config snippets, and personal reflections.

| Week | Focus Area | Link |
|------|------------|------|
| 1️⃣ | Identity & Access Management | [Week 1](./lab-log/week1.md) |
| 2️⃣ | SELinux & Firewall Tuning | [Week 2](./lab-log/week2.md) |
| 3️⃣ | Package Lifecycle & Recovery | [Week 3](./lab-log/week3.md) |

## 📸 Screenshots
Visuals from terminal sessions, AWS dashboard, and config files.  
➡️ [View Screenshots](./screenshots)

## 📓 Lab Log
Detailed notes and command history from each session.  
➡️ [Read Lab Log](./lab-log)

## 🧠 Reflections
Personal takeaways, troubleshooting wins, and lessons learned.  
➡️ [Read Reflections](./reflections)

## 📈 Outcomes
- Reinforced RHCSA exam objectives through real-world practice
- Built a reusable cloud-based Linux lab environment
- Documented procedures for future automation with Ansible

## 🔗 Next Steps
- Expand lab to include multi-node architecture
- Integrate Ansible for configuration management
- Simulate service failures and recovery scenarios
