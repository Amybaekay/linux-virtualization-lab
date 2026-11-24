# Where Things Live — Linux Filesystem Cheat Sheet

**Week 2 | Linux Foundations | Ubuntu Server VM**  


## Purpose

The Filesystem Hierarchy Standard (FHS) describes how Linux directories are organised. Knowing where files belong helps me deploy applications, locate configuration settings, troubleshoot errors and manage storage.

## Standard filesystem locations

| Directory | What lives here | Why I need it in DevOps |
|---|---|---|
| `/` | Root of the filesystem | Starting point for absolute paths |
| `/bin` | Essential commands; often links to `/usr/bin` | Basic system operations |
| `/usr/bin` | Installed executable programs | Locate programs such as `bash`, `python3`, `tar` and `tree` |
| `/sbin`, `/usr/sbin` | System administration programs | Manage users, services, logs and disks |
| `/usr/local/bin` | Locally installed commands | Store locally managed executables |
| `/etc` | System and application configurations | Investigate settings and configuration issues |
| `/home` | Normal users' home directories | Store user files and lab projects |
| `/root` | Root user's home directory | Administrator's personal files |
| `/opt` | Optional/add-on application software | Install or organise custom applications |
| `/var/log` | System and application logs | Investigate events, errors and authentication |
| `/var/lib` | Persistent application state | Locate databases, package records and service state |
| `/var/cache` | Cached application data | Understand regenerable cached files |
| `/srv` | Data provided by services | Organise service data |
| `/tmp` | Temporary files | Store short-lived working files |
| `/run` | Runtime state | Inspect service runtime files |
| `/boot` | Boot-related files | Troubleshoot system startup |
| `/dev` | Device files | Identify and access devices |
| `/proc` | Kernel and process information | Inspect running processes |
| `/sys` | Kernel and hardware interfaces | Inspect device and system attributes |
| `/mnt`, `/media` | Filesystem mount locations | Access mounted storage |

## Locations verified in my Ubuntu lab

| Category | Example | Finding |
|---|---|---|
| Executables | `/usr/bin/python3`, `/usr/bin/tree`, `/usr/bin/tar` | Confirmed with `which` |
| Administration | `/usr/sbin` | Observed utilities such as `groupadd` and `logrotate` |
| Configuration | `/etc/hostname`, `/etc/passwd`, `/etc/netplan` | Identified system settings and account records |
| Logs | `/var/log/auth.log`, `/var/log/syslog` | Examined administrative and system events |
| Persistent data | `/var/lib/apt`, `/var/lib/dpkg`, `/var/lib/plocate/plocate.db` | Identified package state and the filename search database |

## My simulated ShoppingApp

| Location | Purpose |
|---|---|
| `/opt/app/bin/start.sh` | Application startup script |
| `/opt/app/conf/app.conf` | Application settings |
| `/opt/app/conf/config.conf` | Hard link to the same configuration file |
| `/opt/app/logs/app.log` | Simulated application events |
| `/opt/app/logs/small.log` | 5 MiB sample file for size searches |
| `/opt/app/logs/large.log` | 20 MiB sample file for size searches |
| `/opt/app/logs/old.log` | Sample file with an older modification timestamp |
| `/myapp` | Symbolic link pointing to `/opt/app` |
| `/home/app-backup.tar.gz` | Compressed lab backup |

## Quick navigation reminders

- `/` is the root directory; `/root` is the root user's home directory.
- `~` expands to my personal home directory (`/home/amarachi`), not `/home`.
- An absolute path starts at `/`. A relative path starts from the current working directory.
- `.` means the current directory; `..` means the parent directory.
- Hidden files have names beginning with a dot. Use `ls -a` to see them.
- Use `file` before opening unfamiliar files. Binary databases cannot be read meaningfully with `cat`.
- Use `which` for executable locations, `find` for live filesystem searches and `locate` for indexed filename searches.

## Production takeaway

An application may have its executable in `/usr/bin`, configuration in `/etc`, logs in `/var/log` and persistent state in `/var/lib`. My custom lab application groups its sample files under `/opt/app`; real application layouts depend on the software and its installation method.
