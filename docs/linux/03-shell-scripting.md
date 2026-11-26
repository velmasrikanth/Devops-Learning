# Shell Scripting

A shell script is a text file of commands the shell runs in order. Use one when you repeat the same steps: install packages, check a service, or prepare a server.

## A first script

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Host: $(hostname)"
echo "User: $(whoami)"
```

- The first line tells the system to run the file with Bash.
- `set -e` stops the script when a command fails.
- `set -u` stops the script when a variable is unset.
- `set -o pipefail` makes a pipeline fail if any command in it fails.

Make it runnable and start it:

- `chmod +x check-host.sh`
- `./check-host.sh`

## Variables

- `name="web-01"` assigns a value. Do not put spaces around `=`.
- `echo "$name"` prints it. Quote the expansion so spaces stay intact.
- `$1`, `$2` are arguments. `$#` is how many there are.
- `$(date +%F)` runs a command and keeps its output.

```bash
#!/usr/bin/env bash
set -euo pipefail

service="${1:-nginx}"
echo "Checking ${service}"
systemctl is-active "$service"
```

Run it as `./check-service.sh ssh`. If you omit the argument, it checks `nginx`.

## Tests and branches

```bash
if [[ -f /etc/os-release ]]; then
  echo "This machine has an os-release file"
else
  echo "os-release was not found"
fi
```

Useful tests:

- `-f path` is a regular file.
- `-d path` is a directory.
- `-x path` is executable.
- `-z "$var"` is true when the variable is empty.
- `[[ "$a" == "$b" ]]` compares two strings.

## Loops

```bash
for package in curl git vim; do
  sudo apt install -y "$package"
done
```

```bash
while read -r host; do
  echo "Pinging ${host}"
  ping -c 1 "$host"
done < hosts.txt
```

## A safe pattern

- Quote every expansion: `"$var"`, not `$var`.
- Prefer `[[ ]]` over `[ ]`.
- Keep scripts small and named after the job they do.
- Print what the script is about to change before it changes it.
- Do not put passwords in the script. Read them from the environment or a secret store.
