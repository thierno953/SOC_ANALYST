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
<files check_all="yes" report_changes="yes" realtime="yes">/etc/shadow</files>
<files check_all="yes" report_changes="yes" realtime="yes">/etc/group</files>
<files check_all="yes" report_changes="yes" realtime="yes">/etc/passwd</files>
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

![WAZUH](/Wazuh/assets/04.png)

![WAZUH](/Wazuh/assets/05.png)

## On Windows Agent (PowerShell as Administrator):

- Edit the `ossec.conf` (usually in `C:\Program Files (x86)\ossec-agent\ossec.conf`) and **add**:

```sh
<ossec_config>
  <agent_config>
    <localfile>
      <directories check_all="yes" report_changes="yes" realtime="yes">C:\Windows\System32\drivers\etc</directories>
      <directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\Administrator\Desktop\</directories>
      <directories check_all="yes" report_changes="yes" realtime="yes">C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\</directories>
      <registry check_all="yes" report_changes="yes">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run</registry>
    </localfile>
  </agent_config>
</ossec_config>
```

#### Restart the Wazuh agent service:

```sh
Restart-Service -Name wazuh
```

#### Create a test file to trigger monitoring:

```sh
#Linux
echo "new line" >> /root/beta

#CMD
echo test > C:\Users\Administrator\Desktop\test123.txt

#PowerShell
Add-Content -Path "C:\Users\Administrator\Desktop\test123.txt" -Value "new line"
```

![WAZUH](/Wazuh/assets/06.png)

![WAZUH](/Wazuh/assets/07.png)
