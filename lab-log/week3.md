# Week 3 – Automating System Maintenance Tasks 🤖

Shell scripting is hands-down one of the most fun and powerful ways to bend a Linux system to your will. It’s been a minute since I last wrote one, so this week I dusted off the syntax, got my hands dirty, and built a basic maintenance script to kickstart my server automation flow again. Blood’s flowing, logs are rolling, and the terminal’s officially back in business.

## Quick Syntax Refresher 💡😸

### Conditionals

``if [ condition ] then ... elif [ other_condition ] then ... else ... fi``

``while [ condition ] do ... done``

``for label in list do ... done `` → you can use `$(cat list.txt)` or `{0..100}` to avoid typing out each label manually

Multiple Conditions: `[COND1] && [COND2]`, `[COND1] || [COND2]`

### User Input

`read` is the command that waits for user input. `-p` lets you display a prompt message before the user types anything. The value entered by the user gets stored in a variable you specify.

``read -p "Enter your choice: " choice`` → becomes `$choice`
``read -p "Enter mission name: " mission_name`` → becomes `$mission_name`

### String, Number, and File Operators

- Strings: `=`, `!=`  
- Numbers: `-eq`, `-ne`, `-gt`, -`lt`  
- Files: `-e`, `-d`, `-s`, `-x`, `-w`  

### The Importnace of Setting Exit Codes

``echo $?`` → shows last command's exit code 

It's good practice to include exit codes in your scripts to clearly signal success or failure to whatever calls the script—whether it's another script, a cron job, or just you checking the logs. Exit codes help automate decision-making and debugging by letting downstream processes know what happened.

`exit 0` → success  
`exit 1` → error

## My Script 😼

🚧 Come back soon! This section is a work in progress! 🚧

```bash

#!/bin/bash
#
# User & Access Management Automation Script
#
# Features:
# 1. Create a single user with SSH key
# 2. Bulk create users from a CSV (username,ssh_key)
# 3. Disable accounts inactive for X days
# 4. Log all actions to /var/log/user_mgmt.log
#

LOG_FILE="/var/log/user_mgmt.log"

log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') : $1" | sudo tee -a "$LOG_FILE" > /dev/null
}

create_user() {
    USERNAME=$1
    SSH_KEY=$2

    if id "$USERNAME" &>/dev/null; then
        echo "User $USERNAME already exists."
        log_action "User $USERNAME already exists."
    else
        sudo useradd -m -s /bin/bash "$USERNAME"
        echo "User $USERNAME created."
        log_action "User $USERNAME created."
    fi

    HOME_DIR="/home/$USERNAME"
    SSH_DIR="$HOME_DIR/.ssh"

    sudo mkdir -p "$SSH_DIR"
    echo "$SSH_KEY" | sudo tee "$SSH_DIR/authorized_keys" > /dev/null
    sudo chmod 700 "$SSH_DIR"
    sudo chmod 600 "$SSH_DIR/authorized_keys"
    sudo chown -R "$USERNAME:$USERNAME" "$SSH_DIR"

    echo "SSH key added for $USERNAME."
    log_action "SSH key added for $USERNAME."
}

bulk_create_users() {
    CSV_FILE=$1

    if [ ! -f "$CSV_FILE" ]; then
        echo "CSV file not found: $CSV_FILE"
        exit 1
    fi

    tail -n +2 "$CSV_FILE" | while IFS=, read -r USERNAME SSH_KEY
    do
        create_user "$USERNAME" "$SSH_KEY"
    done
}

disable_inactive_users() {
    DAYS=$1
    if [ -z "$DAYS" ]; then
        echo "Please specify number of days."
        exit 1
    fi

    echo "Disabling accounts inactive for $DAYS days..."
    sudo usermod --expiredate 1 root &>/dev/null # safety check (root should never be disabled)

    # Find users inactive for given days and lock them
    sudo lastlog -b "$DAYS" | awk 'NR>1 && $2 != "" {print $1}' | while read -r USERNAME
    do
        if [ "$USERNAME" != "root" ]; then
            sudo usermod --expiredate 1 "$USERNAME"
            echo "Disabled inactive user: $USERNAME"
            log_action "Disabled inactive user: $USERNAME"
        fi
    done
}

show_help() {
    echo "Usage:"
    echo "  $0 single <username> \"<ssh-key>\"     # Create single user"
    echo "  $0 bulk users.csv                     # Bulk create users from CSV"
    echo "  $0 disable <days>                      # Disable accounts inactive for X days"
    echo "  $0 help                                # Show this help message"
}

# -----------------------------
# Main
# -----------------------------

ACTION=$1
shift

case "$ACTION" in
    single)
        create_user "$1" "$2"
        ;;
    bulk)
        bulk_create_users "$1"
        ;;
    disable)
        disable_inactive_users "$1"
        ;;
    help|*)
        show_help
        ;;
esac

```
