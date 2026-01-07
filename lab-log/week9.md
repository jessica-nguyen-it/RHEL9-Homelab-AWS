# Week 9 - Scheduling Tasks with Cron, Anacron, and At

Wouldn’t it be great if our machines could help manage themselves?  

For example, imagine being able to schedule backups, automatically monitor service health, detect critical failures, flag unauthorized login attempts, and get alerts the moment something needs your attention.


This week, I learned how to do exactly that! :^) 

## Cron

A useful tool for automating repetitive tasks that need to run at specific intervals.

The system‑wide configuration file lives at /etc/crontab, but instead of modifying the global file, you should update your own user‑level crontab instead. For example:

```bash
# opens your personal cron table
crontab -e

# shows your current crontab entries
crontab -l

# shows the root user's crontab entries
sudo crontab -l

# modify or delete another user's crontab
sudo crontab -e -u RockLee
sudo crontab -r -u RockLee

```

#### Syntax
NOTE: This helpful excerpt is pulled directly from the KodeKloud website!

```
In a cron job, the first five fields define the schedule:

    - Minute: 0–59
    - Hour: 0–23 (0 indicates midnight, 23 indicates 11 PM)
    - Day of the Month: 1–31
    - Month: 1–12 (or short month names such as jan, feb, etc.)
    - Day of the Week: 0–6 (0 or 7 represents Sunday; short names are also allowed, e.g., mon, tue)

You can modify these fields using special characters:

    - An asterisk (*) represents all possible values.
    - A comma (,) separates multiple values.
    - A dash (-) specifies a range (e.g., “2-4” means 2, 3, and 4).
    - A slash (/) represents step values, for example, “*/4” runs every 4 units.
```

```bash
# Locate full path of command you intend to automate
which touch
/usr/bin/touch

# Schedule the 'touch' command to run daily at 6:35 AM
35 6 * * * /usr/bin/touch test_passed
```

#### Using special Cron Directories
Cron also supports scheduling through special directories, where any executable script you place will run automatically at the associated interval.

- /etc/cron.daily
- /etc/cron.hourly
- /etc/cron.weekly
- /etc/cron.monthly

For example, to schedule a shell script to run hourly:
```bash 
# Create the script
touch shellscript

# Copy it to the hourly directory with root privileges
sudo cp shellscript /etc/cron.hourly/

# Set the correct permissions for execution
sudo chmod +rx /etc/cron.hourly/shellscript

# Delete the script when no longer needed
sudo rm /etc/cron.hourly/shellscript

```

#### Verifying Cron Jobs

1. To view user-specific cron logs, execute `sudo cat /var/log/cron`

	- These log entries display the execution time, command details, and output. If you wish to view only executed command logs, filter the logs using: `sudo grep CMD /var/log/cron`
2. To view system-wide log entries, execute `cat /etc/crontab`

## Anacron

Anacron is a simple tool that runs scheduled jobs without requiring the system to be on 24/7.

Unlike cron, which skips tasks if the machine was powered off, if a scheduled job is missed, it will simply execute after the system restarts.

```bash
# Schedule a job every 3 days with a 10-minute delay
sudo vim /etc/anacrontab

#period in days    delay in minutes    job-identifier    command
1                   5                   cron.daily        nice run-parts
7                   25                  cron.weekly       nice run-parts
@monthly            45                  cron.monthly      nice run-parts /etc/cron.monthly
3                   10                  test_job          /usr/bin/touch /root/anacron_created_this

```

Why is there a 10‑minute delay? It helps stagger execution so that if multiple jobs are scheduled for the same day, they don’t all run at once. Since Anacron’s smallest scheduling unit is a day, this delay prevents everything from firing simultaneously and overloading the system.

Also, since it's impractical to wait a whole day for testing purposes. you can manually trigger your Anacron jobs with `sudo anacron -n`.

#### Verifying Anacron Jobs

1. To view anacron logs, simply filter for the anacron log with `sudo grep anacron /var/log/cron`
2. You can also alter the command to enhance job logging:

```bash 
# Update the test_job line 
1       10          test_job         /bin/echo "Testing anacron" | systemd-cat --identifier=test_job

# Force anacron to run all jobs
sudo anacron -n -f

# View the system log
journalctl -e

# You should notice that it's easier to identify your job within the entries!

```


## At

At lets you run a job once at a specific time in the future.  

Unlike cron or Anacron, which repeat on a schedule, at is for one‑off tasks, like “run this script tomorrow at 3 PM” or “execute this command in 10 minutes.”

```bash
# Enter the at command
at 15:00

# Enter the command you want to execure
/usr/bin/touch file_created_by_at

# Press Enter on an empty line then press Ctrl-D to save
```

Here are some more common examples of time options:
```bash
$ at 'August 20 2022'
$ at '2:30 August 20 2022'
$ at 'now + 30 minutes'
$ at 'now + 3 hours'
$ at 'now + 3 days'
$ at 'now + 3 weeks'
$ at 'now + 3 months'
```

And lastly, managing At jobs:
```bash
# View scheduled jobs
atq
20      Wed Nov 17 08:30:00 2021 a aaron

# View a specific job
at -c 20

# Removing a scheduled job
atrm 20
```

#### Verifying At Jobs

1. Just like with Cron and Anacron logs, simply filter for the at log with `sudo grep at /var/log/cron` 
2. Also like Anacron, to see more detailed output, pipe it's output to `systemd-cat` using `systemd-cat --identifier=at_scheduled_backup`
	- Once the jpb executes, you can check it using `journalctl | grep at_scheduled_backup` 