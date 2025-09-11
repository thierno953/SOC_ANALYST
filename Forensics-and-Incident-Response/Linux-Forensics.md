# Linux Forensics

- Linux forensics is the process of analyzing a Linux-based system to detect, investigate, and respond to security incidents.

- **Objectives**: Evidence Collection, Incident Detection and Response, Analysis, Documentation & Reporting.

- Provides the ability to **reconstruct user actions**, detect malware, and support legal or internal investigations.

## Types of Evidence

```sh
| Evidence Type        | Tools / Examples                                                |
| -------------------- | --------------------------------------------------------------- |
| Live data collection | EDR (business tools), `ps`, `top`, `htop`, `netstat`, `lsof -i` |
| Memory Dump          | `dd`, RAM capture, LiME (Linux Memory Extractor)                |
| Disk Imaging         | `dd`                                                            |
| File System Analysis | Metadata (`ls -l`), deleted files                               |
| Log Files            | System logs (`/var/log`), application logs                      |
| Config Files         | `/etc/passwd`, `/etc/shadow`                                    |
| User Activity        | `.bash_history`, `.zsh_history`                                 |
| Login History        | `/var/log/lastlog`, `/var/log/auth.log`                         |
| Scheduled Tasks      | `/etc/cron*`, `crontab -l`                                      |
```

# System Log Analysis

## Using `journalctl` (systemd logs)

```sh
journalctl -n 100                   # Show last 100 logs
journalctl --since "1 hour ago"     # Logs from the last hour
journalctl -u ssh                   # Logs from SSH service
journalctl | grep "error"           # Filter logs containing "error"
journalctl -b > logs_last_boot.txt  # Export logs from last boot
```

## Parsing logs with `awk`

- `awk` is a powerful text-processing language used for data extraction and reporting. It is particularly useful for parsing log files and extracting specific fields or patterns.

```sh
awk '{print $1, $2, $3}' /var/log/syslog                 # Print date/time fields
awk '/root/ {print $0}' /var/log/syslog                 # Filter lines containing root
awk '/failed/ {count++} END {print count}' /var/log/auth.log  # Count failed logins
awk '/session opened/ {user[$11]++} END {for (u in user) print u, user[u]}' /var/log/auth.log
```

## Real-time monitoring with `tail`

- `tail` is a command-line utility used to display the end of a text file or stream. It is commonly used to monitor log files in real-time.

```sh
tail -f /var/log/syslog
tail -f /var/log/syslog | grep --color=auto "error"
tail -f /var/log/syslog /var/log/auth.log
tail -n 50 -f /var/log/syslog
```

## Summarizing logs with `logwatch`

- `logwatch` is a log analysis tool that generates detailed summaries of system logs. It provides insights into system activity and potential issues by aggregating and summarizing log entries.

```sh
sudo apt-get install logwatch
sudo logwatch --detail low --range today
```

## Tracking system events with `auditd`

- Use `Auditd` to set up and analyze audit rules for tracking specific system events.

```sh
sudo apt-get install auditd
sudo systemctl start auditd
sudo auditctl -w /bin/chmod -p x -k chmod_changes
sudo chmod 755 warning_logs.txt
sudo ausearch -k chmod_changes
```

## Linux File System Analysis

- **Purpose**: Reconstruct events, identify security breaches, recover deleted data, ensure compliance, and gather legal evidence.

- **Activities**:

  - Files & directories: Attributes (`lsattr, chattr`), timestamps (stat), hidden files (`ls -a, find / -name ".\*"`)

  - Metadata examination

  - Content analysis: Malware detection (`grep`, YARA, ClamAV, VirusTotal), AES-256 encrypted data

  - Advanced: Data recovery, rootkit or malware analysis

## Linux File System Overview

```sh
| Concept               | Notes                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| Types                 | Ext2, Ext3, Ext4, ReiserFS, XFS, Btrfs                                |
| Structure             | Superblock, inode table, data blocks, directories, metadata           |
| Important Directories | `/home`, `/etc`, `/var/log`, `/var/www`, `/etc/passwd`, `/etc/shadow` |
| Common Tools          | `ls`, `stat`, `find`, `grep`, `file`, `df`, `du`, `mount`             |
```

## File & Directory Analysis

```sh
ls -l /home
ls -la /home
stat /home/user/file.txt
file /home/user/file.txt
find /home -name "*.txt"
find /home -empty
find /home -type f -size +100M
find /home -type f -mtime -7
find /home -type f -atime -7
find /home -type f -ctime -7
```

## Disk Usage

```sh
df -h
du -sh /home/user
du -ah /home/user
```

## Search file content

```sh
echo "This is a test file for malware analysis." > test.exe
grep "malware" *.exe
grep -r "malware" /home/user
grep -c "malware" *.exe
```
