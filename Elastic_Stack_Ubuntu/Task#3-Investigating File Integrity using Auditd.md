# Investigating File Integrity Using Auditd

#### Objective

- Monitor the integrity of critical files (e.g., `/etc/passwd`, `/etc/shadow`) to detect suspicious changes on a Linux host using **Auditd**, and integrate it with **ELK SIEM** for centralized analysis.

## Installing Auditd

- **Auditd** is a security auditing service built into Linux, used to** monitor critical file integrity and detect any suspicious activity** on a host. In this project, it's integrated with **ELK SIEM** to visualize and analyze events in real time, enhancing our ability to respond quickly to potential compromises.

```sh
apt update
sudo apt install auditd audispd-plugins -y
systemctl enable auditd
systemctl start auditd
systemctl status auditd
```

- Check the logs:

```sh
tail -f /var/log/audit/audit.log
```

## Preparing ELK for Detection

- In **Kibana** -> `Management -> Integrations`

- Search for `Auditd Logs` -> Add the integration

- Set file paths: `/var/log/audit/audit.log*`

- Assign to an existing **Agent Policy** -> Continue -> Confirm

- Available Dashboards:

  - `[Logs Auditd] Audit Events`

## Simulating an Attack and Viewing Events

- Edit the audit rules:

```sh
nano /etc/audit/rules.d/audit.rules
```

- Example content:

```sh
## First rule - delete all existing rules
-D

## Increase buffers to handle stress events
-b 8192

## How long to wait for bursts of events
--backlog_wait_time 60000

## Set failure mode to log to syslog
-f 1

## Monitor sensitive files
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
```

-Restart the service:

```sh
systemctl restart auditd
```

- Simulate a change to `/etc/passwd`:

```sh
# Add a user
adduser linuxuser
passwd linuxuser

# Manually modify the file (simulate an attack)
echo "testuser:x:1001:1001::/home/testuser:/bin/bash" >> /etc/passwd
```

#### Viewing Logs

```sh
tail -f /var/log/audit/audit.log
ausearch -k passwd_changes
```

- View locally using `ausearch`

- View in Kibana -> `Analytics > Discover`

```sh
auditd.log.key:"passwd_changes"
```

```sh
message: "/etc/passwd"
```

## Incident Response

- Remove the manually added line (or comment it out):

```sh
nano /etc/passwd

# Remove or comment out:
# testuser:x:1001:1001::/home/testuser:/bin/bash
```
