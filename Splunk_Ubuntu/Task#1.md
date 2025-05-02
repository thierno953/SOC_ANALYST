# Task#1: Detection of Unauthorized Access on Linux Blocked by Fail2Ban

- Install Fail2Ban
- Set up Fail2Ban
- Simulate the attack
- Analyze

```sh
root@node01:~# apt update
```

#### Install and set up Fail2ban

```sh
root@node01:~# apt install fail2ban -y
```

```sh
root@node01:~# nano /etc/fail2ban/jail.local
```

```sh
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

```sh
root@node01:~# /opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
root@node01:~# systemctl restart fail2ban
root@node01:~# fail2ban-client status
root@node01:~# tail -f /var/log/fail2ban.log
```

```sh
root@node01:~# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = false
index = security_incidents
sourcetype = linux_secure
whitelist = Failed|invalid|Denied

[monitor:///var/log/fail2ban.log]
disabled = false
sourcetype = fail2ban
index = fail2ban_logs
```

```sh
root@node01:~# /opt/splunkforwarder/bin/splunk restart
```

#### Simulate the SSH Brute force attack

```sh
root@attack:~# apt update
root@attack:~# apt install hydra -y
root@attack:~$ hydra -l admin -P passwords.txt <IP_VICTIM_MACHINE> ssh
```

```sh
root@node01:~#  tail -f /var/log/fail2ban.log
```

#### Analyzing logs on Splunk

`Search & Reporting`

```sh
index="fail2ban_logs"
index="fail2ban_logs" src="<IP_ATTACK_MACHINE>"
index="fail2ban_logs" sourcetype="fail2ban" src="<IP_ATTACK_MACHINE>"
index="fail2ban_logs" | search "<IP_ATTACK_MACHINE>"
```
