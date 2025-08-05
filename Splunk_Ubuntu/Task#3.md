# Sensitive File Integrity Monitoring

- Install and configure **auditd** to monitor sensitive files

- Configure sending logs to Splunk

- Simulate an attack (unauthorized modification)

- Analyze logs and respond to the incident

#### Installing and configuring auditd

```sh
apt-get update
apt-get install auditd -y
systemctl start auditd
systemctl status auditd
```

#### Configure integrity rules:

```sh
nano /etc/audit/rules.d/audit.rules
```

#### Content of audit.rules

```sh
## First rule - delete all existing rules
-D

## Increase buffers to handle burst events (increase for busy systems)
-b 8192

## Time to wait during event bursts
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1

## Watch for changes in /etc/
-w /etc/ -p wa -k file_integrity
```

#### Apply the rules

```sh
systemctl restart auditd
auditctl -l   # Verify that the rules are loaded
```

## Configure Splunk Universal Forwarder

- Edit `inputs.conf`

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Content of `inputs.conf`

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
index = linux_file_integrity
sourcetype = auditd

[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

#### Restart the Splunk Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulate an attack (unauthorized modification)

- Create or modify a file in `/etc/`

```sh
nano /etc/myfirstfile.txt

# File content
This is my first file.
```

#### Monitor logs in real time

```sh
tail -f /var/log/audit/audit.log
```

#### Search logs related to the key `file_integrity`

```sh
ausearch -k file_integrity
ausearch -k file_integrity | grep myfirstfile
```

#### Search queries in Splunk `Search & Reporting`

- Use these queries in Splunk UI to analyze the events

```sh
index=linux_file_integrity sourcetype=auditd "myfirstfile"
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_12.png)

```sh
index=linux_file_integrity sourcetype=auditd key=file_integrity
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_13.png)
