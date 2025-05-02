# Task#3: File Integrity Monitoring for Sensitive files

- Installing Auditd
- Setting up Splunk
- Simulate attack & Analyze
- Incident Response

#### Installing and setting up Auditd

```sh
root@node01:~# apt-get update
root@node01:~# apt-get install auditd -y
root@node01:~# systemctl start auditd
root@node01:~# systemctl status auditd
root@node01:~# cd /etc/audit/rules.d
```

```sh
root@node01:/etc/audit/rules.d# nano audit.rules
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

-w /etc/ -p wa -k file_integrity
```

```sh
root@node01:/etc/audit/rules.d# cd ../../..
root@node01:/# systemctl restart auditd
root@node01:/# sudo auditctl -l
```

#### Setting up Splunk

```sh
root@node01:/# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = auditd
index = linux_file_integrity

[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

```sh
root@node01:/# /opt/splunkforwarder/bin/splunk restart
```

#### Simulate Unauthorized change attempt & Analyzing logs on Splunk

```sh
root@node01:/# nano /etc/myfirstfile.txt

#This is my first file.
```

```sh
root@node01:/# tail -f /var/log/audit/audit.log
```

```sh
root@node01:/# ausearch -k file_integrity
root@node01:/# ausearch -k file_integrity | grep myfirstfile
```

`Search & Reporting`

```sh
index=linux_file_integrity sourcetype=auditd
index=linux_file_integrity sourcetype=auditd "myfirstfile"
index=linux_file_integrity sourcetype=auditd key=file_integrity
```
