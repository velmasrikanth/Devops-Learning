# Linux Basics

Linux is the operating system most servers, containers, and CI runners use. These notes cover the commands you need before Docker, Kubernetes, and shell scripts.

## What you are working with

- Kernel: talks to CPU, memory, disks, and the network.
- Shell: the program that reads your commands. Bash is the usual one.
- Distribution: a packaged Linux. Ubuntu and RHEL-family systems are the ones you will see most often.
- Everything is a file: processes, disks, and settings are reached through paths.

## Where you are

- `pwd` prints the current directory.
- `ls -la` lists files, including hidden ones, with permissions and owners.
- `cd /path` moves into a directory. `cd -` returns to the previous one.
- `cd ~` or `cd` alone returns to your home directory.

Absolute paths start at `/`. Relative paths start from where you are now.

## Looking at files

- `cat file` prints a whole file.
- `less file` opens a file you can scroll. Press `q` to leave.
- `head -n 20 file` and `tail -n 20 file` show the start or the end.
- `tail -f /var/log/syslog` follows a log as new lines are written.

## Finding things

- `which nginx` shows the program that will run when you type that name.
- `find /etc -name "*.conf"` searches by file name.
- `grep -R "Listen" /etc/nginx` searches file contents.
- `history` shows commands you already ran.

## Processes

- `ps aux` lists running processes.
- `top` shows live CPU and memory use. Press `q` to leave.
- `kill PID` asks a process to stop. `kill -9 PID` forces it.
- `systemctl status ssh` shows whether a service is running.

## Packages

Ubuntu and Debian:

- `sudo apt update` refreshes the package index.
- `sudo apt install nginx` installs a package.
- `sudo apt remove nginx` removes it.

RHEL, CentOS, and Fedora:

- `sudo dnf install nginx`
- `sudo dnf remove nginx`

## Getting help

- `man ls` opens the manual page.
- `ls --help` prints a short usage summary.
- `type cd` tells you whether a name is a builtin, an alias, or a file.
