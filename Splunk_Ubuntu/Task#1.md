# Unauthorized Access Detection with Fail2Ban

- Detect unauthorized SSH access on Linux using **Fail2Ban**, with logs sent to **Splunk** via the **Universal Forwarder**.

## Install and Configure Fail2Ban on the Target Machine

#### Update and install Fail2Ban

```sh
apt update && apt install fail2ban -y
```

#### Configure the SSH jail (`jail.local`)

```sh
nano /etc/fail2ban/jail.local
```

- Content of the file:

```sh
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

#### Restart and check Fail2Ban status

```sh
systemctl restart fail2ban
fail2ban-client status
tail -f /var/log/fail2ban.log
```

## Configure the Splunk Universal Forwarder to Collect Logs

#### Add monitoring of the log files

```sh
/opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
```

#### Edit `inputs.conf` to include Fail2Ban and SSH logs

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Content of the file:

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
index = fail2ban_logs
sourcetype = fail2ban
```

#### Restart the Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulate an SSH Brute Force Attack

#### On the attacking machine, install Hydra

```sh
apt update && apt install hydra -y
```

#### Launch an SSH brute force attack

```sh
hydra -l admin -P password.txt <IP_VICTIME> ssh
```

- Make sure `admin` is an existing user and the SSH port is reachable.

#### Observe logs on the victim machine

```sh
tail -f /var/log/fail2ban.log
```

## Event Analysis in Splunk

#### Search queries in Splunk's Search & Reporting App:

```sh
index="fail2ban_logs" sourcetype="fail2ban" | search "192.168.129.239"
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_09.png)

```sh
index="security_incidents" sourcetype="linux_secure" "Failed password"
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_10.png)
