# Task#2: Detecting Suspicious activities using SysmonForLinux

- Install SysmonForLinux
- Prepare ELK for detection
- Simulate the Attack and visualize the events
- Incident Response

#### Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# 1. Register Microsoft key and feed
root@fleet-agent:~# wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
root@fleet-agent:~# sudo dpkg -i packages-microsoft-prod.deb

# 2. Install SysmonForLinux
root@fleet-agent:~# sudo apt-get update
root@fleet-agent:~# sudo apt-get install sysmonforlinux
```

```sh
root@fleet-agent:~# nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
root@fleet-agent:~# sysmon -i sysmon-config.xml
```

```sh
root@fleet-agent:~# systemctl restart sysmon
root@fleet-agent:~# systemctl status sysmon

root@fleet-agent:~# tail -f /var/log/syslog | grep sysmon
```

#### Prepare ELK for detection

`Management > Integrations (search Sysmon for linux and added)`

```sh
Paths: /var/log/syslog*
Existing hosts > select: Agent policies > Save and continue
Assets; Dashboards: [Sysmon] Sysmon for Linux Logs Overview
```

- `Analystics > Discover`

```sh
process.name:sysmon
```

#### Simulate the attack and visualize the events

- [MalwareBazaar](https://bazaar.abuse.ch/)

```sh
root@fleet-agent:~# touch /etc/apt/apt.conf.d/99-suspicious-config
root@fleet-agent:~# sudo bash -c "echo 'malicious config' > /etc/apt/apt.conf.d/99-malicious-config"
root@fleet-agent:~# curl -X GET "https://bazaar.abuse.ch/" -v
```

- `Analystics > Discover`

```sh
process.name:sysmon and message:"99"
process.name:sysmon and message:"bazaar.abuse.ch"
```

#### Incident Response

```sh
root@fleet-agent:~# find / -name "99-malicious-config" -type f
root@fleet-agent:~# rm -rf /etc/apt/apt.conf.d/99-suspicious-config
root@fleet-agent:~# rm -rf /etc/apt/apt.conf.d/99-malicious-config
root@fleet-agent:~# find / -name "99-malicious-config" -type f
root@fleet-agent:~# netstat -ltnp
```
