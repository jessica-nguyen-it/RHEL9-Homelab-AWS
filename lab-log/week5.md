# Week 5 - Processes, Logs, and TuneD

This week I dug into process management, log analysis, and TuneD. Process management and log analysis are big ones that I used extensively during my time as an SRE intern. TuneD, on the other hand, is completely new to me (yay!) 

NOTE: Starting this week these notes will be in cheat‑sheet style to keep things concise.

## Process Management

It is very important for anyone working with Linux systems to be able to diagnose and manage processes. This includes understanding how they are created, monitoring them, and terminating them when necessary.


#### Viewing Processes

```bash
ps → snapshot of current terminal processes
ps aux → all processes with details (USER, PID, %CPU, %MEM, COMMAND)
pgrep -a <name> → search processes by name
ps fax → visualize parent–child process relationships
```


#### Real Time Monitoring

```bash
top → real-time process monitoring
Scroll with arrows/Page Up/Down, quit with q
```

#### Priority & Niceness

Niceness ranges from -20 (the highest priority) to +19 (the lowest priority). Niceness refers to how well a process “plays” with other processes on the system. A higher niceness value means the process is more willing to yield CPU time to others, while a lower niceness value means it demands more priority. In essence, the “nicer” a process is, the less it competes with other programs for resources.

```bash
nice -n <value> command → configure command to start with a certain niceness value
renice <value> <PID> → change niceness of an already running process
```

### Signals

You can send signals to running processes to control their behavior. Signals allow you to pause, resume, or terminate a process, as well as trigger specific actions defined by the program. For example, SIGKILL forcefully stops a process, SIGTERM requests a graceful shutdown, and SIGHUP can be used to reload configuration files. These signals are typically sent using commands like kill, pkill, or killall.

```bash
kill -<signal> <PID> → send signal (default: SIGTERM)
```

Some common signals inlcude:

- SIGKILL → force terminate
- SIGSTOP → pause
- SIGCONT → resume

#### Job Control

```bash
    Ctrl+Z → suspend foreground process
    fg [%job] → bring job to foreground
    bg [%job] → resume job in background
    jobs → list active jobs
```

#### Open Files

Processes don’t just consume CPU and memory, they also interact with files on the system. Being able to see which files a process has opened, or which process is holding onto a particular file, is crucial for diagnosing resource locks, debugging issues, and understanding system behavior.

```bash
    lsof -p <PID> → list files opened by process
    lsof <file> → see which process is using a file
```


## Log Analysis

Another important skill is knowing which log files to examine and how to sort through them to determine if something is going wrong, what specifically is failing, and to gather hints on how you can fix the issue.

#### Log Locations

    System logs: /var/log/messages → general events, boot info, service activity

    Authentication & security: /var/log/secure → SSH logins, sudo usage, password changes

    Boot logs: /var/log/boot.log (and rotated versions with dates) → startup sequence details

    Audit logs: /var/log/audit → SELinux and security auditing

    Service-specific logs: directories like /var/log/httpd, /var/log/firewalld, /var/log/libvirt → application/service events

    User activity: last and lastlog commands → session history, login attempts

#### Live Monitoring

    tail -F → follow logs in real time, useful for troubleshooting active issues

    journalctl -f → systemd journal live mode, structured and service-aware

    Use cases: watching SSH login attempts, monitoring application startup, tracking errors as they occur

#### journalctl Basics

Journalctl is another big one!

On systemd‑based systems, journalctl acts as a unified log viewer that brings together messages from across the system. Unlike traditional flat log files, it allows you to filter by command path, service unit, or even by boot session, making it easier to zero in on the exact context of an issue. 

Each entry comes with structured metadata such as timestamps, priorities, and service names, which helps you quickly understand not just what happened but when and why. 

And, if persistent storage is configured under /var/log/journal/, logs are kept across reboots, giving you a continuous history to work with rather than losing data each time the system restarts.

#### Filtering & Analysis

    By priority: debug, info, notice, warning, err, crit, alert, emerg

    By time window: -S (since) and -U (until) for precise ranges

    By boot session: -b 0 (current boot), -b -1 (previous boot)

    By keyword/regex: -g option for pattern matching inside logs

## TuneD

Finally, the last topic for this week: TuneD

TuneD is essentially a performance optimization service that monitors system activity and applies predefined profiles. These profiles fine‑tune system behavior to either conserve power or boost performance, depending on the workload. One of its strengths is the ability to automatically select the most appropriate profile based on the environment, whether the system is running as a server, a virtual machine, or in general desktop use.

#### A couple profile types

1. Power-saving profiles: reduce energy use by disabling inactive devices or enabling low-power modes.

2. Performance-boosting profiles: enhance throughput, reduce latency, optimize for virtualization or HPC workloads.

3. Balanced profile: default equilibrium between performance and power consumption.

#### Profile Locations

`/usr/lib/tuned` → distribution-provided profiles (each with its own tuned.conf).

`/etc/tuned` → main config (tuned-main.conf) and custom profiles (override defaults).

#### Managing Profiles (tuned-adm)

```bash
tuned-adm list → list all available profiles and show which one is active

tuned-adm active → display only the currently active profile

tuned-adm profile <name> → switch to a specific profile (e.g., balanced, throughput-performance)

tuned-adm profile_info → show detailed information about the current or specified profile

tuned-adm recommend → suggest the best profile for the current environment

tuned-adm verify → check if system settings match the active profile

tuned-adm auto_profile → enable automatic profile selection and monitoring

tuned-adm --help → display help and usage options for all commands
```

#### Profile Operations

1. Switching: move between profiles depending on workload (e.g., VM vs. high-throughput server).

2. Merging: combine multiple profiles (last one wins if conflicts occur).

#### Dynamic vs. Static Tuning

1. Static tuning: applies chosen profile settings as-is.

2. Dynamic tuning: adjusts settings on the fly based on system demand.

   - Controlled via /etc/tuned/tuned-main.conf.
   - Disabled by default (dynamic_tuning = 0).
   - Enable by setting dynamic_tuning = 1 and restarting TuneD.

#### Practical Use Cases

So... what are some examples of the practical use cases?

- On virtual machines, TuneD defaults to the virtual‑guest profile to optimize for virtualization. 
- On high‑throughput servers, the throughput‑performance profile is used to maximize speed and efficiency. 
- For general systems, the balanced profile provides a middle ground between performance and power saving. 
- Finally, for specialized workloads such as HPC, low‑latency networking, or desktop environments, targeted profiles are available to fine‑tune system behavior.

Fin!
