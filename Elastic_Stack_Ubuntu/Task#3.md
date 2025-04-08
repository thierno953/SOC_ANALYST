# Task#3: Investigating File Integrity using Auditd

- Objective
- Install Auditd
- Prepare Elk for detection
- Simulate the attack and visualize the events
- Incident Response

#### Install Auditd

```sh
root@fleet-agent:~# apt update
root@fleet-agent:~# sudo apt install auditd audispd-plugins -y
root@fleet-agent:~# systemctl enable auditd
root@fleet-agent:~# systemctl start auditd
root@fleet-agent:~# systemctl status auditd
```

```sh
root@fleet-agent:~# nano /etc/audit/rules.d/audit.rules
```

```sh
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1
```

```sh
root@fleet-agent:~# tail -f /var/log/audit/audit.log
```

#### Prepare ELK for detection

`Management > Integrations > Add Auditd Logs`

```sh
(Paths: /var/log/audit/audit.log* > and select existing hosts and save continue)
(Auditd Logs --> assets [Logs Auditd] Audit Events)
```

#### Simulate the attack and visualize the events

```sh
root@fleet-agent:~# adduser linuxuser
root@fleet-agent:~# passwd linuxuser
root@fleet-agent:~# less /etc/passwd
root@fleet-agent:~# nano /etc/audit/rules.d/audit.rules
```

```sh
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1

-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
```

```sh
root@fleet-agent:~# systemctl restart auditd
root@fleet-agent:~# sudo echo "testuser:x:1001:1001::/home/testuser:/bin/bash" >> /etc/passwd
root@fleet-agent:~# tail -f /var/log/audit/audit.log
root@fleet-agent:~# ausearch -k passwd_changes
```

- `Analystics > Discover`

```sh
auditd.log.key:"passwd_changes"

"/etc/passwd"
```

#### Incident Response

```sh
root@fleet-agent:~# nano /etc/passwd 

#testuser:x:1001:1001::/home/testuser:/bin/bash
```
