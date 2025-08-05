# User Account Activity Monitoring

- Install Sysmon for Linux

- Configure log ingestion with Splunk

- Simulate malicious activity

- Visualize events in Splunk

- Respond to the incident

#### Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# Add Microsoft repository
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install sysmonforlinux -y
```

#### Create and apply a Sysmon configuration file

```sh
# Download or create a custom XML config file
nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
# Apply the configuration
sysmon -i sysmon-config.xml

# Check service status
systemctl restart sysmon
systemctl status sysmon
```

#### Configure Splunk Universal Forwarder

```sh
# Check for sysmon logs in syslog
grep -i sysmon /var/log/syslog
```

```sh
# Edit Splunk input configuration
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

```sh
# Restart Splunk Forwarder
/opt/splunkforwarder/bin/splunk restart
```

#### Simulate malicious activity

```sh
# Create a fake user
adduser maluser --disabled-password --gecos ""
echo "maluser:Password123!" | chpasswd
```

```sh
# Monitor traces in syslog and auth.log
tail -f /var/log/syslog | grep maluser
tail -f /var/log/auth.log | grep maluser
```

#### Search queries in Splunk `Search & Reporting`

```sh
index="linux_os_logs" process=sysmon maluser
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_15.png)

## Incident Response

- Verify and remove the malicious account

```sh
less /etc/passwd
sudo chage -l maluser
sudo deluser --remove-home maluser
```
