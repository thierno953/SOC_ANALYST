# Security Investigation of an Ubuntu Machine using Splunk

- [Download Splunk Enterprise 9.4.3](https://www.splunk.com/en_us/download/splunk-enterprise.html)

## Splunk Enterprise Installation (Server Side)

#### Update and Download

```sh
apt update
wget -O splunk-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/9.4.3/linux/splunk-9.4.3-237ebbd22314-linux-amd64.deb"
chmod +x splunk-9.4.3-237ebbd22314-linux-amd64.deb
dpkg -i splunk-9.4.3-237ebbd22314-linux-amd64.deb
```

#### Enable Autostart + Firewall Rules

```sh
/opt/splunk/bin/splunk enable boot-start
ufw enable
ufw allow OpenSSH
ufw allow 8000     # Web UI
ufw allow 9997     # Receiving logs from Forwarder
ufw status
```

#### Start Splunk

```sh
/opt/splunk/bin/splunk start --accept-license
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_02.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_03.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_04.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_05.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_06.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_07.png)

## Installing Splunk Universal Forwarder (Client Side)

- [Download Splunk Universal Forwarder 9.4.3](https://www.splunk.com/en_us/download/universal-forwarder.html)

#### Download and Install

```sh
wget -O splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.3/linux/splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb"
chmod +x splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
dpkg -i splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
```

#### Enable Boot Start and Start Service

```sh
/opt/splunkforwarder/bin/splunk enable boot-start
/opt/splunkforwarder/bin/splunk start
```

#### Connect to Splunk Enterprise Server

- Replace `<Splunk_Server_IP>` with your actual Splunk Enterprise server IP address:

```sh
/opt/splunkforwarder/bin/splunk add forward-server <IP_Splunk_Entreprise>:9997 -auth admin:Admin@123
/opt/splunkforwarder/bin/splunk list forward-server
```

#### Firewall Settings

```sh
ufw enable
ufw allow OpenSSH
ufw allow 9997
ufw status
```

#### Recommended Permissions

```sh
chown -R splunkfwd:splunkfwd /opt/splunkforwarder
chmod -R 755 /opt/splunkforwarder
```

#### Monitor System Logs (`/var/log/syslog`)

```sh
tail -f /var/log/syslog
```

#### Add File Monitoring to Forwarder

```sh
/opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
```

#### Advanced Configuration with `inputs.conf`

- Create or Edit the `inputs.conf` file

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Sample Content

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

#### Restart the Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

#### Query in Splunk (Search & Reporting)

```sh
index="linux_os_logs" sourcetype=syslog
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_08.png)
