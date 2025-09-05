**Objective**:

- Detect and investigate user account creation, modification, and deletion events on an Ubuntu server using **Sysmon for Linux** and **Splunk**, simulate suspicious activity, and respond appropriately.

- **Tools**: Sysmon for Linux, Splunk Universal Forwarder, Splunk

### Step 1: Install Sysmon for Linux

- Add Microsoft key and repository:

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
```

### 2. Install SysmonForLinux

```sh
sudo apt-get update
sudo apt-get install sysmonforlinux
```

#### Create Sysmon configuration:

```sh
nano sysmon-config.xml
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

### Install Sysmon with your config:

```sh
sudo sysmon -i sysmon-config.xml
sudo systemctl restart sysmon
sudo systemctl status sysmon
```

### Step 2: Set Up Splunk Universal Forwarder

- Check Sysmon logs:

```sh
cd /var/log/
grep -i sysmon syslog
```

### Configure Splunk UF to monitor syslog:

```sh
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

### Restart Splunk Forwarder:

```sh
sudo /opt/splunkforwarder/bin/splunk restart
```

- **Optional**: Install **Splunk Add-on for Sysmon for Linux** to enhance field extraction and visualization.

### Step 3: Simulate Suspicious User Account Activity

- Create a test user:

```sh
sudo adduser maluser
```

### Monitor account-related logs in real-time:

```sh
tail -f /var/log/syslog | grep maluser
```

### Step 4: Search & Visualize in Splunk

- Search all Sysmon events:

```sh
index="linux_os_logs" sysmon
```

![Splunk](/Splunk_Ubuntu/assets/10.png)

### Filter for processes related to account creation:

```sh
index="linux_os_logs" process=sysmon maluser
```

![Splunk](/Splunk_Ubuntu/assets/11.png)

### Step 5: Incident Response

- Check user details:

```sh
less /etc/passwd
chage -l maluser
```

### Delete unauthorized user:

```sh
sudo deluser maluser
```

### Confirm deletion:

```sh
grep maluser /etc/passwd
```
