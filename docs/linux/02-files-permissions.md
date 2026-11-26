# Files and Permissions

Linux decides who can read, change, or run a file from its owner, its group, and the permission bits.

## Create, copy, move, delete

- `mkdir -p app/logs` creates a directory and any missing parents.
- `touch app/notes.txt` creates an empty file, or updates the timestamp if it exists.
- `cp source.txt backup.txt` copies a file. Add `-r` to copy a directory.
- `mv old.txt new.txt` renames or moves a file.
- `rm file.txt` deletes a file. `rm -r dir` deletes a directory. There is no trash.

## How permissions read

`ls -l` prints a line like `-rwxr-xr--`.

- First character: `-` for a file, `d` for a directory.
- Next three: owner. Here the owner can read, write, and execute.
- Next three: group. Here the group can read and execute.
- Last three: everyone else. Here others can only read.

The same bits as numbers:

- `4` is read, `2` is write, `1` is execute.
- `7` is `4+2+1`, so read, write, and execute.
- `755` means owner `rwx`, group `r-x`, others `r-x`.
- `644` means owner `rw-`, and everyone else `r--`.

## Change them

- `chmod 644 app.conf` sets exact bits.
- `chmod +x script.sh` adds execute for everyone.
- `chown user:group file` changes the owner and group.
- `sudo` runs one command as root. Use it for system paths such as `/etc` and `/usr`.

## Paths you will use often

- `/home/user` is a person's home directory.
- `/etc` holds system configuration.
- `/var/log` holds logs.
- `/tmp` is temporary and is cleared across reboots on many systems.
- `/usr/bin` and `/usr/local/bin` hold programs.

## Links

- `ln -s /etc/nginx/nginx.conf ~/nginx.conf` creates a symbolic link.
- The link points at a path. Deleting the link does not delete the target.
- A hard link is a second name for the same file. It cannot point at a directory.
