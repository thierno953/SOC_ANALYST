# Unauthorized Access Detection with Fail2Ban & Splunk

- Detect unauthorized SSH access on Linux using **Fail2Ban**, with logs sent to **Splunk** via the **Universal Forwarder**.

## Install and Configure Fail2Ban on the Target Machine

```sh
apt update && apt install fail2ban -y
systemctl restart fail2ban
fail2ban-client status sshd
ls /etc/fail2ban/filter.d/recidive.conf
```

#### Configure the `jail.local` file

```sh
nano /etc/fail2ban/jail.local
```

- Content:

```sh
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 <TON_IP_FIXE>
dbfile = /var/lib/fail2ban/fail2ban.sqlite3
logtarget = SYSLOG

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600

[recidive]
enabled  = true
filter   = recidive
logpath  = /var/log/fail2ban.log
bantime  = 604800
findtime = 86400
maxretry = 5
action   = iptables-allports[name=recidive]
```

## Restart and check Fail2Ban status

```sh
sudo systemctl restart fail2ban
sudo fail2ban-client reload
sudo fail2ban-client status
sudo fail2ban-client status recidive
tail -f /var/log/fail2ban.log
```

## Configure Splunk Universal Forwarder

```sh
/opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
/opt/splunkforwarder/bin/splunk list index
/opt/splunkforwarder/bin/splunk list monitor
```

#### Edit `inputs.conf`

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Content:

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
host = server-ssh-prod

[monitor:///var/log/auth.log]
disabled = false
index = security_incidents
sourcetype = linux_secure
whitelist = Failed password|Invalid user|authentication failure
host = server-ssh-prod

[monitor:///var/log/fail2ban.log]
disabled = false
index = fail2ban_logs
sourcetype = fail2ban
host = ubuntu-prod-ssh
```

## Restart Splunk Forwarder:

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulate an SSH Brute Force Attack

- Install Hydra:

```sh
apt update && apt install hydra -y
```

#### Test attack:

```sh
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://<IP_VICTIME>
```

## Monitor Fail2Ban Activity

```sh
sudo fail2ban-client status sshd
sudo fail2ban-client status recidive
tail -f /var/log/fail2ban.log
watch -n 1 "fail2ban-client status sshd | grep 'Banned IP list'"
```

#### Basic search:

```sh
index="fail2ban_logs" sourcetype="fail2ban"
```

#### Failed SSH login attempts:

```sh
index="security_incidents" sourcetype="linux_secure" "Failed password"
```

#### Banned IPs with count:

```sh
(index="fail2ban_logs" OR index="security_incidents")
| rex field=_raw "Ban\s(?<banned_ip>\d+\.\d+\.\d+\.\d+)"
| stats count AS attempts BY banned_ip
| sort -attempts
```

#### Timeline of bans:

```sh
index="fail2ban_logs"
| rex field=_raw "Ban\s(?<banned_ip>\d+\.\d+\.\d+\.\d+)"
| timechart span=1h count BY banned_ip
```
