# Task#5: Monitoring User Account Activity

- Install Sysmon for Linux
- Setting up Splunk
- Simulate attack & visualize
- Incident Response

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

#### Setting up Splunk UF and Splunk Dashboard

```sh
root@forwarder:~# cd /var/log/
root@forwarder:/var/log# grep -i sysmon syslog
```

```sh
root@forwarder:/var/log# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

`Find More Apps > Browse More Apps > Splunk Add-on for Sysmon for Linux`

```sh
root@forwarder:/var/log# /opt/splunkforwarder/bin/splunk restart
```

####Search & Reporting

```sh
index="linux_os_logs"
index="linux_os_logs" sysmon
index="linux_os_logs" process=sysmon
```

#### Simulate the account activities and Visualizing it on Splunk

```sh
root@forwarder:~# adduser maluser
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
```

```sh
root@forwarder:~# tail -f /var/log/syslog | grep maluser
```

####Search & Reporting

```sh
index="linux_os_logs" process=sysmon maluser
```

#### Incident Response

```sh
root@forwarder:~# less /etc/passwd
root@forwarder:~# chage -l maluser
root@forwarder:~# deluser maluser
```
