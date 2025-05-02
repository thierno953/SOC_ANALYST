# Security Investigation of Ubuntu Machine using Splunk

#### Download and Install Splunk

- [Splunk Enterprise Previous Releases](https://www.splunk.com/en_us/download/previous-releases.html?locale=en_us)

![Enterprise](/assets/splunk_linux_01.png)

```sh
root@splunk:~# apt update
root@splunk:~# wget -O splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.3.1/linux/splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
root@splunk:~# chmod +x splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@splunk:~# dpkg -i splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@splunk:~# /opt/splunk/bin/splunk enable boot-start

root@splunk:~# ufw enable
root@splunk:~# ufw allow OpenSSH
root@splunk:~# ufw allow 8000
root@splunk:~# ufw allow 9997
root@splunk:~# ufw status

root@splunk:~# /opt/splunk/bin/splunk start
```

![Enterprise](/assets/splunk_linux_02.png)
![Enterprise](/assets/splunk_linux_03.png)
![Enterprise](/assets/splunk_linux_04.png)
![Enterprise](/assets/splunk_linux_05.png)
![Enterprise](/assets/splunk_linux_06.png)
![Enterprise](/assets/splunk_linux_07.png)

#### Setting up Ubuntu Machine with Splunk Universal Forwarder

- Install UF on Linux
- Set up UF
- Enable Forwarding & Receiving on Splunk Dashboard

- [Splunk Universal Forwarder Previous Releases](https://www.splunk.com/en_us/download/previous-releases-universal-forwarder.html)

```sh
root@node01:~# wget -O splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.3.1/linux/splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
root@node01:~# chmod +x splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@node01:~# dpkg -i splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@node01:~# /opt/splunkforwarder/bin/splunk enable boot-start
root@node01:~# /opt/splunkforwarder/bin/splunk start
root@node01:~# /opt/splunkforwarder/bin/splunk add forward-server <IP_Splunk_Entreprise>:9997 -auth admin:Admin@123
root@node01:~# /opt/splunkforwarder/bin/splunk list forward-server

root@node01:~# ufw enable
root@node01:~# ufw allow OpenSSH
root@node01:~# ufw allow 8000
root@node01:~# ufw allow 9997
root@node01:~# ufw status

root@node01:~# chown -R splunkfwd:splunkfwd /opt/splunkforwarder
root@node01:~# chmod -R 755 /opt/splunkforwarder
root@node01:~# /opt/splunkforwarder/bin/splunk restart
```

#### Creating Index for Syslog

```sh
root@node01:~# cd /var/log/
root@node01:/var/log# tail -f syslog

root@node01:/var/log# /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
root@node01:/var/log# cd /opt/splunkforwarder/etc/system/local/
root@node01:/opt/splunkforwarder/etc/system/local# ls
outputs.conf  README  server.conf
root@node01:/opt/splunkforwarder/etc/system/local#
```

```sh
root@node01:/opt/splunkforwarder/etc/system/local# nano inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

```sh
root@node01:/opt/splunkforwarder/etc/system/local# cd ../../../..
root@node01:/opt# /opt/splunkforwarder/bin/splunk restart
```

`Settings > Indexes > New Index`

`Settings > Source types > New Source Type`

`Search & Reporting`

```sh
index="linux_os_logs"
index="linux_os_logs" sourcetype=syslog
```
