# Detecting DNS Queries with Sysmon + Wazuh on Windows

#### Install Sysmon on Windows

- Download Sysmon: [https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

- Use the community config (or create your own): [https://github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)

#### Install Sysmon with config (PowerShell):

```sh
cd C:\Users\Administrator\Desktop\Sysmon\
.\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula
```

![WAZUH](/Wazuh/assets/11.png)

#### Configure Wazuh Agent on Windows

- Ensure the Wazuh agent on Windows is monitoring Sysmon logs:
  - Edit the Wazuh agent config (`C:\Program Files (x86)\ossec-agent\ossec.conf`) and add:

```sh
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

#### Restart Wazuh agent service:

```sh
Restart-Service wazuh
```

#### Generate DNS Event

- Run a DNS query to test detection:

```sh
nslookup wazuh.com
```

#### Create Custom Rule in Wazuh Manager

- On the **Wazuh Manager**, create the custom rule file:

```sh
sudo nano /var/ossec/ruleset/rules/0999-custom_sysmon_rules.xml
```

- Paste your custom rules:

```sh
<group name="custom_sysmon_rules">

  <!-- Base rule for all Sysmon events -->
  <rule id="61600" level="0">
    <field name="win.system.providerName">Microsoft-Windows-Sysmon</field>
    <description>Base rule for Sysmon events</description>
  </rule>

  <!-- DNS Query (Event ID 22) -->
  <rule id="61650" level="8" overwrite="yes">
    <if_sid>61600</if_sid>
    <field name="win.system.eventID">22</field>
    <description>Sysmon - DNS query detected: $(win.eventdata.Query)</description>
    <options>no_full_log</options>
    <group>sysmon, dns, windows</group>
  </rule>


  <!-- Process Creation (Event ID 1) -->
  <rule id="61651" level="10">
    <if_sid>61600</if_sid>
    <field name="win.system.eventID">1</field>
    <description>Sysmon - Process created: $(win.eventdata.Image)</description>
    <group>sysmon, process_creation, windows</group>
  </rule>

  <!-- Network Connection Detected (Event ID 3) -->
  <rule id="61652" level="7">
    <if_sid>61600</if_sid>
    <field name="win.system.eventID">3</field>
    <description>Sysmon - Network connection detected: $(win.eventdata.DestinationIp):$(win.eventdata.DestinationPort)</description>
    <group>sysmon, network_connection, windows</group>
  </rule>

  <!-- Process Injection (Event ID 10) -->
  <rule id="61653" level="12">
    <if_sid>61600</if_sid>
    <field name="win.system.eventID">10</field>
    <description>Sysmon - Process Injection detected from $(win.eventdata.SourceImage) to $(win.eventdata.TargetImage)</description>
    <group>sysmon, process_injection, windows</group>
  </rule>

  <!-- Suspicious PowerShell Process -->
  <rule id="61654" level="15">
    <if_sid>61651</if_sid>
    <field name="win.eventdata.Image">C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe</field>
    <description>Sysmon - Suspicious process launched (PowerShell)</description>
    <group>sysmon, suspicious, windows</group>
  </rule>

</group>
```

#### Restart Wazuh Manager

```sh
sudo systemctl restart wazuh-manager
```

```sh
powershell.exe -Command "Get-Process"
```

#### Verify in Wazuh Dashboard

- Go to **Wazuh Web UI** -> **Security Events**.

- Search for:

```sh
agent.name:"win22" AND (rule.id:61650 OR rule.id:61651 OR rule.id:61652 OR rule.id:61653 OR rule.id:61654)
```

![WAZUH](/Wazuh/assets/12.png)
