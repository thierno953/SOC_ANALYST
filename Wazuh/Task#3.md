# Wazuh File Integrity Monitoring (FIM) setup steps, both for Linux and Windows agents

- Reference: [https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html](https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html)

## On Linux Agent (agent01):

- Edit the Wazuh configuration:

```sh
sudo nano /var/ossec/etc/ossec.conf
```

#### Add the following lines for File Integrity Monitoring (FIM):

```sh
<!-- FIM Linux -->
<directories check_all="yes" report_changes="yes" realtime="yes">/root</directories>
<directories check_all="yes" report_changes="yes" realtime="yes">/etc/shadow</directories>
<directories check_all="yes" report_changes="yes" realtime="yes">/etc/group</directories>
<directories check_all="yes" report_changes="yes" realtime="yes">/etc/passwd</directories>
```

#### Restart the Wazuh agent:

```sh
sudo systemctl restart wazuh-agent
```

#### Create a test file to monitor changes:

```sh
nano /root/beta
```

- Content

```sh
This is my Under monitoring
```

## On Windows Agent (PowerShell as Administrator):

- Edit the `ossec.conf` (usually in `C:\Program Files (x86)\ossec-agent\ossec.conf`) and **add**:

```sh
<directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\Administrator\Desktop</directories>
```

#### Restart the Wazuh agent service:

```sh
Restart-Service -Name wazuh
```

#### Create a test file to trigger monitoring:

```sh
echo test > C:\Users\Administrator\Desktop\test123.txt
```
