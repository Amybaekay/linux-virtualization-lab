# 30 Linux Commands Practised

**Week 2 | Linux Foundations | Ubuntu Server VM**  

This reference documents 30 command examples from my filesystem and navigation exercises. Repeated utilities with different options count as separate practice examples because they perform different operations. Commands that change files are lab-only examples; do not rerun them blindly on an existing installation.

| No. | Command | What it does | Lab application |
|---:|---|---|---|
| 1 | `pwd` | Prints the current directory | Confirm my location before working |
| 2 | `cd /opt` | Changes to an absolute directory | Navigate to application storage |
| 3 | `cd ..` | Moves to the parent directory | Navigate back one level |
| 4 | `ls -a` | Lists all entries, including hidden ones | Inspect `/opt` before creating the application |
| 5 | `ls -la /opt/app/logs` | Shows detailed listings including hidden entries | Examine ownership, permissions and timestamps |
| 6 | `ls -lh /opt/app/logs` | Shows detailed listings with readable sizes | Confirm the sizes of sample logs |
| 7 | `sudo mkdir -p /opt/app/{bin,conf,logs}` | Creates the parent and three subdirectories | Prepare the application structure |
| 8 | `sudo touch /opt/app/bin/start.sh` | Creates an empty file if absent | Create the startup script file |
| 9 | `sudo nano /opt/app/conf/app.conf` | Opens a file for editing | Enter sample configuration settings |
| 10 | `cat /opt/app/conf/app.conf` | Displays text file contents | Verify application configuration |
| 11 | `sudo mv -i linux.txt networking.txt aws.text study/` | Moves files, prompting before overwrite | Move the original study files from `/home` into `/home/study` |
| 12 | `sudo apt install tree` | Installs a package and required dependencies | Install the directory visualisation tool |
| 13 | `tree /opt/app` | Displays a directory hierarchy | Verify application folders and files |
| 14 | `file /etc/os-release` | Identifies the file type | Discover a symbolic link |
| 15 | `which python3` | Finds an executable in PATH | Confirm Python is in `/usr/bin` |
| 16 | `sudo fallocate -l 5M /opt/app/logs/small.log` | Allocates a 5 MiB sample file | Prepare size-based search data |
| 17 | `find /opt/app/logs -type f -size -113c` | Finds regular files smaller than 113 bytes | Locate the 112-byte application log |
| 18 | `sudo touch -d '15 days ago' /opt/app/logs/old.log` | Sets an older timestamp on a sample file | Prepare modification-time search data |
| 19 | `find /opt/app/logs -type f -mtime +7` | Finds files at least eight full days old | Identify the old sample log |
| 20 | `find /opt/app/logs -type f -mtime -1` | Finds files modified within the past 24 hours | Identify recently modified logs |
| 21 | `sudo updatedb` | Refreshes the locate database | Index the newly created files |
| 22 | `locate app.conf` | Searches indexed paths for a name | Locate the application configuration |
| 23 | `sudo ln /opt/app/conf/app.conf /opt/app/conf/config.conf` | Creates a hard link | Give the configuration a second name |
| 24 | `ls -li /opt/app/conf` | Shows inode numbers and metadata | Verify that the hard links share an inode |
| 25 | `sudo ln -s /opt/app /myapp` | Creates a symbolic link | Provide a shortcut to the application directory |
| 26 | `sudo tar -czvf /home/app-backup.tar.gz -C /opt app` | Archives and compresses the application | Create a separate backup archive |
| 27 | `tar -tzvf /home/app-backup.tar.gz` | Lists archive contents without extraction | Inspect the backup before restoration |
| 28 | `sudo tar -xzvpf /home/app-backup.tar.gz -C /home/app-restore` | Extracts while retaining file permissions | Restore to a separate destination (must already exist) |
| 29 | `sudo tail -n 10 /var/log/auth.log` | Displays the last ten log lines | Inspect authentication and administrative activity |
| 30 | `sudo tail -n 10 /var/log/syslog` | Displays the last ten system log lines | Inspect system and service events |

## Key lessons

- Linux commands and options are case-sensitive. `find -size` uses lowercase `k`, uppercase `M` and `G`, or `c` for bytes; it does not accept decimal quantities.
- `ls -lh` displays each file's apparent size. The `total` line reflects allocated blocks, which is different from the file's apparent size.
- `find` searches the live filesystem; `locate` uses a database that may need refreshing.
- A hard link shares the original file's inode and data; a symbolic link points to a path. Neither is an independent backup.
- `tar` archives files; gzip compression reduces the archive size. `-p` on extraction preserves permissions.
- `sudo` runs an authorised administrative command. Use it only when necessary, and keep personal practice files under `~` where possible.

## Outcome

Practised filesystem navigation, file creation, metadata inspection, file searching, links, package installation, backup and restoration, and examined actual Linux system logs. These exercises help to understand how software is organised and maintained on a Linux server.

