# Sensitive File Integrity Monitoring

- Install and configure **auditd** to monitor sensitive files

- Configure sending logs to Splunk

- Simulate an attack (unauthorized modification)

- Analyze logs and respond to the incident

## Install and Configure auditd

```sh
apt-get update
apt-get install auditd -y
systemctl start auditd
systemctl enable auditd
systemctl status auditd
```

## Configure File Integrity Rules

- Edit `/etc/audit/rules.d/audit.rules`:

```sh
nano /etc/audit/rules.d/audit.rules
```

#### Content of audit.rules

```sh
# Delete all existing rules
-D

# Increase buffers for burst events
-b 8192

# Wait time during event bursts
--backlog_wait_time 60000

# Set failure mode to syslog
-f 1

# Monitor sensitive directories and files
-w /etc/ -p wa -k file_integrity
-w /etc/passwd -p wa -k file_integrity
-w /etc/shadow -p wa -k file_integrity
```

## Apply Rules

```sh
systemctl restart auditd
auditctl -l   # Verify active rules
```

## Configure Splunk Universal Forwarder

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Add monitors:

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

## Parse Fields

- Edit `transforms.conf`:

```sh
nano /opt/splunkforwarder/etc/system/local/transforms.conf
```

```sh
[auditd_file_exe]
REGEX = (?:name|exe)=(".*?"|\S+)
FORMAT = file::$1

[auditd_user]
REGEX = auid=(\d+|-1)
FORMAT = user::$1

[auditd_pid]
REGEX = pid=(\d+)
FORMAT = pid::$1

[auditd_key]
REGEX = key=(\S+)
FORMAT = key::$1
```

- Edit `props.conf`:

```sh
nano /opt/splunkforwarder/etc/system/local/props.conf
```

```sh
[auditd]
REPORT-auditd_fields = auditd_file_exe, auditd_user, auditd_pid, auditd_key
```

#### Restart the forwarder:

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulate an Unauthorized Modification

```sh
echo "unauthorized change" | sudo tee -a /etc/myfirstfile.txt
```

## Check audit logs:

```sh
tail -f /var/log/audit/audit.log
ausearch -k file_integrity | tail -20
ausearch -k file_integrity | grep myfirstfile
```

## Analyze in Splunk

```sh
index=linux_file_integrity sourcetype=auditd "myfirstfile"

index=linux_file_integrity sourcetype=auditd key=file_integrity
| table _time host user exe file
| sort -_time

index=linux_file_integrity sourcetype=auditd
| table file user host _raw
| head 20

index=linux_file_integrity sourcetype=auditd
| table _time host user file pid exe
| head 20

index=linux_file_integrity sourcetype=auditd
| eval file=coalesce(file, "unknown"), user=coalesce(user, "unknown"), host=coalesce(host, "unknown"), pid=coalesce(pid, "unknown"), exe=coalesce(exe, "unknown")
| stats count by file user host pid exe
| sort -count
```
