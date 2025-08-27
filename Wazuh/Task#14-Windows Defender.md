# Windows Defender Integration with Wazuh

### Configure the Wazuh Agent (Windows)

- Edit the Windows agent configuration file (`ossec.conf`) to collect Windows Defender logs and enable Active

```sh
<ossec_config>
  <!-- Monitor Windows Defender events -->
  <localfile>
    <location>Microsoft-Windows-Windows Defender/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <!-- Active Response: Kill malicious processes on the agent -->
  <active-response>
    <command>windows-kill-process</command>
    <location>agent</location>
    <rules_id>100202,100203</rules_id>
    <timeout>600</timeout>
  </active-response>
</ossec_config>
```

### Restart the Windows agent

```sh
net stop wazuh-agent
net start wazuh-agent
```

### Test Detection with the EICAR File

- Download the EICAR test file (used to simulate malware detection without risk):

```sh
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

- Alternatively, create the file manually:

```sh
echo "X50!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*" > "$env:USERPROFILE\Downloads\eicar.com.txt"
```

### Create a Custom Decoder

- On the **Wazuh manager**, define a decoder to extract relevant Windows Defender fields:

```sh
nano /var/ossec/etc/decoders/local_decoder.xml
```

```sh
<decoder name="windows-defender">
  <parent>windows_eventchannel</parent>
  <prematch>Microsoft Defender Antivirus</prematch>

  <regex>product Name:(.+)</regex>
  <order>product_name</order>

  <regex>threat Name:(.+)</regex>
  <order>threat_name</order>

  <regex>action Name:(.+)</regex>
  <order>action_name</order>

  <regex>path:file:_(.+)</regex>
  <order>file_path</order>

  <regex>process Name:(.+)</regex>
  <order>process_name</order>

  <regex>User:(.+)</regex>
  <order>user</order>

  <regex>severity Name:(.+)</regex>
  <order>severity</order>
</decoder>
```

### Define Local Rules

- Create rules to trigger alerts based on Windows Defender events:

```sh
nano /var/ossec/etc/rules/local_rules.xml
```

```sh
<!-- Local rules for Windows Defender -->

<!-- Product detection -->
<group name="windows,defender,normalization">
  <rule id="100201" level="3">
    <decoded_as>windows-defender</decoded_as>
    <field name="product_name">Microsoft Defender Antivirus</field>
    <description>Windows Defender detected product name</description>
  </rule>
</group>

<!-- Malware detection -->
<group name="windows_defender,antivirus,malware">
  <rule id="100202" level="10">
    <decoded_as>windows-defender</decoded_as>
    <field name="threat_name">.*</field>
    <description>Windows Defender detected malware or unwanted software</description>
  </rule>
</group>

<!-- Actions taken by Defender -->
<group name="windows_defender,actions">
  <rule id="100203" level="5">
    <decoded_as>windows-defender</decoded_as>
    <field name="action_name">Quarantine|Remove|Allowed|Blocked|Not Applicable</field>
    <description>Windows Defender performed an action on a file or threat</description>
  </rule>
</group>
```

### Restart Wazuh Manager

```sh
systemctl restart wazuh-manager
systemctl status wazuh-manager
```
