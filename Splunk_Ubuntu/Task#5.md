# User Account Activity Monitoring

- Install Sysmon for Linux

- Configure log ingestion with Splunk

- Simulate malicious activity

- Visualize events in Splunk

- Respond to the incident

#### Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install sysmonforlinux -y
```

```sh
sysmon -h
systemctl status sysmon
```

#### Create and apply a Sysmon configuration file

```sh
nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
<Sysmon schemaversion="1.0">

  <!-- Surveillance des créations de fichiers suspects -->
  <FileCreate onmatch="include">
    <TargetFilename condition="contains">/home/</TargetFilename>
    <TargetFilename condition="contains">/tmp/</TargetFilename>
    <TargetFilename condition="end with">.sh</TargetFilename>
    <TargetFilename condition="end with">.exe</TargetFilename>
    <TargetFilename condition="contains">/var/log/auth.log</TargetFilename>
    <TargetFilename condition="contains">/var/log/syslog</TargetFilename>
  </FileCreate>

  <!-- Surveillance des processus suspects -->
  <ProcessCreate onmatch="include">
    <CommandLine condition="contains">/tmp/</CommandLine>
    <CommandLine condition="end with">.sh</CommandLine>
    <CommandLine condition="contains">curl</CommandLine>
    <CommandLine condition="contains">wget</CommandLine>
    <CommandLine condition="contains">nc</CommandLine>
    <CommandLine condition="contains">nmap</CommandLine>
    <CommandLine condition="contains">rm -rf</CommandLine>
    <CommandLine condition="contains">chmod 777</CommandLine>
    <CommandLine condition="contains">chown</CommandLine>
  </ProcessCreate>

  <!-- Détection du chargement de bibliothèques malveillantes -->
  <ImageLoad onmatch="include">
    <ImageLoaded condition="contains">/tmp/</ImageLoaded>
    <ImageLoaded condition="contains">/dev/shm/</ImageLoaded>
  </ImageLoad>

  <!-- Surveillance réseau -->
  <NetworkConnect onmatch="include">
    <DestinationIp condition="excludes">127.0.0.1</DestinationIp>
    <DestinationIp condition="excludes">::1</DestinationIp>
  </NetworkConnect>

  <!-- Commandes sensibles -->
  <CommandLine onmatch="include">
    <CommandLine condition="contains">sudo</CommandLine>
    <CommandLine condition="contains">useradd</CommandLine>
    <CommandLine condition="contains">adduser</CommandLine>
    <CommandLine condition="contains">passwd</CommandLine>
  </CommandLine>

</Sysmon>
```

```sh
sudo sysmon -c sysmon-config.xml
sudo systemctl restart sysmon
```

```sh
grep -i sysmon /var/log/syslog
```

```sh
sudo nano /etc/rsyslog.conf
```

```sh
if $programname == 'sysmon' then /var/log/sysmon.log
& stop
```

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = false
index = linux_os_logs
sourcetype = authlog

[monitor:///var/log/sysmon.log]
disabled = false
index = linux_os_logs
sourcetype = sysmon
```

```sh
nano /opt/splunkforwarder/etc/system/local/props.conf
```

```sh
[sysmon]
EXTRACT-pid = ProcessId=(\d+)
EXTRACT-ppid = ParentProcessId=(\d+)
EXTRACT-user = User=(\S+)
EXTRACT-command = CommandLine="([^"]+)"
```

```sh
# Restart Splunk Forwarder
/opt/splunkforwarder/bin/splunk restart
```

#### Simulate malicious activity

```sh
sudo adduser maluser --disabled-password --gecos ""
echo "maluser:Password123!" | sudo chpasswd
echo "echo 'Malicious script executed'" > /tmp/test.sh
chmod +x /tmp/test.sh
/tmp/test.sh
ssh localhost
```

- Monitor traces in logs:

```sh
sudo tail -f /var/log/syslog | grep maluser
sudo tail -f /var/log/auth.log | grep maluser
sudo tail -f /var/log/sysmon.log
```

#### Search queries in Splunk `Search & Reporting`

```sh
index="linux_os_logs" sourcetype=sysmon CommandLine="*useradd*" OR CommandLine="*adduser*"
index="linux_os_logs" sourcetype=sysmon FileCreate="*.sh" OR FileCreate="*.exe"
index="linux_os_logs" sourcetype=sysmon CommandLine="*chmod 777*" OR CommandLine="*rm -rf*"
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_15.png)

## Incident Response

- Verify and remove the malicious account

```sh
sudo chage -l maluser
grep maluser /etc/passwd
sudo deluser --remove-home maluser
rm -rf /tmp/test.sh
```
