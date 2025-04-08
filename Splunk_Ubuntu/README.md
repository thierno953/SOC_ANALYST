# Security Investigation of Ubuntu Machine using Splunk

#### Download and Install Splunk

- [Splunk Enterprise ](https://www.splunk.com/en_us/download/splunk-enterprise.html?locale=en_us)

```sh
root@entreprise:~# sudo apt update
root@entreprise:~# wget -O splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.3.1/linux/splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
root@entreprise:~# sudo chmod +x splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@entreprise:~# sudo dpkg -i splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@entreprise:~# sudo /opt/splunk/bin/splunk enable boot-start

root@entreprise:~# sudo ufw enable
root@entreprise:~# sudo ufw allow ssh
root@entreprise:~# sudo ufw allow OpenSSH
root@entreprise:~# sudo ufw allow 8000
root@entreprise:~# sudo ufw allow 9997
root@entreprise:~# sudo ufw status

root@entreprise:~# sudo /opt/splunk/bin/splunk start
```

#### Setting up Ubuntu Machine with Splunk Universal Forwarder

- Install UF on Linux
- Set up UF
- Enable Forwarding & Receiving on Splunk Dashboard

- [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)

```sh
root@forwarder:~# wget -O splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.3.1/linux/splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
root@forwarder:~# sudo chmod +x splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@forwarder:~# sudo dpkg -i splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
root@forwarder:~# sudo /opt/splunkforwarder/bin/splunk enable boot-start
root@forwarder:~# sudo /opt/splunkforwarder/bin/splunk start
root@forwarder:~# sudo /opt/splunkforwarder/bin/splunk add forward-server <IP_Splunk_Entreprise>:9997 -auth admin:Admin@123
root@forwarder:~# sudo /opt/splunkforwarder/bin/splunk list forward-server

root@forwarder:~# sudo ufw enable
root@forwarder:~# sudo ufw allow ssh
root@forwarder:~# sudo ufw allow OpenSSH
root@forwarder:~# sudo ufw allow 8000
root@forwarder:~# sudo ufw allow 9997
root@forwarder:~# sudo ufw status

root@forwarder:~# chown -R splunkfwd:splunkfwd /opt/splunkforwarder
root@forwarder:~# chmod -R 755 /opt/splunkforwarder
root@forwarder:~# /opt/splunkforwarder/bin/splunk restart
```

#### Creating Index for Syslog

```sh
root@forwarder:~# cd /var/log/
root@forwarder:~# tail -f syslog

root@forwarder:~# /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
root@forwarder:~# cd /opt/splunkforwarder/etc/system/local/
```

```sh
root@forwarder:~# nano inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

```sh
root@forwarder:/opt/splunkforwarder/etc/system/local# cd ../../../..
root@forwarder:/opt# /opt/splunkforwarder/bin/splunk restart
```

####Search & Reporting

```sh
index="linux_os_logs"
index="linux_os_logs" sourcetype=syslog
```
