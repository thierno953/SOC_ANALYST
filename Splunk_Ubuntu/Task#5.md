# Task#5: Monitoring User Account Activity

- Install Sysmon for Linux
- Setting up Splunk
- Simulate attack & visualize
- Incident Response

#### Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# 1. Register Microsoft key and feed
root@node01:~# wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
root@node01:~# sudo dpkg -i packages-microsoft-prod.deb

# 2. Install SysmonForLinux
root@node01:~# apt-get update
root@node01:~# apt-get install sysmonforlinux
```

```sh
root@node01:~# nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
root@node01:~# sysmon -i sysmon-config.xml
```

```sh
root@node01:~# systemctl restart sysmon
root@node01:~# systemctl status sysmon
```

#### Setting up Splunk UF and Splunk Dashboard

```sh
root@node01:~# cd /var/log/
root@node01:/var/log# grep -i sysmon syslog
```

```sh
root@node01:/var/log# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

#[monitor:///var/log/audit/audit.log]
#disabled = false
#sourcetype = auditd
#index = linux_file_integrity

#[monitor:///var/log/suricata/fast.log]
#disabled = false
#index = network_security_logs
#sourcetype = suricata
```

`Find More Apps > Browse More Apps > Splunk Add-on for Sysmon for Linux`

```sh
root@node01:/var/log# /opt/splunkforwarder/bin/splunk restart
```

####Search & Reporting

```sh
index="linux_os_logs"
index="linux_os_logs" sysmon
index="linux_os_logs" process=sysmon
```

#### Simulate the account activities and Visualizing it on Splunk

```sh
root@node01:~# adduser maluser
Adding user `maluser' ...
Adding new group `maluser' (1002) ...
Adding new user `maluser' (1002) with group `maluser' ...
Creating home directory `/home/maluser' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
No password has been supplied.
New password:
Retype new password:
No password has been supplied.
New password:
Retype new password:
No password has been supplied.
passwd: Authentication token manipulation error
passwd: password unchanged
Try again? [y/N]
Changing the user information for maluser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
root@node01:~# 
```

```sh
root@node01:~# tail -f /var/log/syslog | grep maluser
```

####Search & Reporting

```sh
index="linux_os_logs" process=sysmon maluser
```

#### Incident Response

```sh
root@node01:~# less /etc/passwd
root@node01:~# chage -l maluser
root@node01:~# deluser maluser
```
