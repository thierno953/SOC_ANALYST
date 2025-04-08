# Task#3: File Integrity Monitoring for Sensitive files

- Installing Auditd
- Setting up Splunk
- Simulate attack & Analyze
- Incident Response

#### Installing and setting up Auditd

```sh
root@forwarder:$ apt-get update
root@forwarder:$ apt-get install auditd -y
root@forwarder:$ systemctl start auditd
root@forwarder:$ systemctl status auditd
root@forwarder:$ cd /etc/audit/rules.d
```

```sh
root@forwarder:/etc/audit/rules.d# nano audit.rules
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
root@forwarder:/etc/audit/rules.d# cd ../../..
```

```sh
root@forwarder:/# sudo systemctl restart auditd
root@forwarder:/# sudo auditctl -l
```

#### Setting up Splunk

```sh
root@forwarder:/# nano /opt/splunkforwarder/etc/system/local/inputs.conf
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
root@forwarder:/# /opt/splunkforwarder/bin/splunk restart
```

#### Simulate Unauthorized change attempt & Analyzing logs on Splunk

```sh
root@forwarder:~# nano /etc/myfirstfile.txt

#This is my first file.
```

```sh
root@forwarder:/# tail -f /var/log/audit/audit.log
```

```sh
root@forwarder:~# ausearch -k file_integrity
root@forwarder:~# ausearch -k file_integrity | grep myfirstfile
```

####Search & Reporting

```sh
index=linux_file_integrity sourcetype=auditd
index=linux_file_integrity sourcetype=auditd "myfirstfile"
index=linux_file_integrity sourcetype=auditd key=file_integrity
```
