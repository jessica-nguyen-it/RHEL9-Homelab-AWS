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
