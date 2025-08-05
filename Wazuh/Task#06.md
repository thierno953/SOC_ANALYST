# Detecting DNS Queries with Sysmon + Wazuh on Windows

#### Install Sysmon on Windows

- Download Sysmon: [https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

- Use the community config (or create your own): [https://github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)

#### Minimal config for DNS queries (sysmonconfig-export.xml):

```sh
<EventFiltering>
  <DnsQuery onmatch="include">
    <Image condition="contains">svchost.exe</Image>
  </DnsQuery>
</EventFiltering>
```

#### Install Sysmon with config (PowerShell):

```sh
cd C:\Users\Administrator\Desktop\Sysmon\
.\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula
```

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
  <!-- Match Event ID 22 -->
  <rule id="255001" level="8">
    <if_sid>61600</if_sid>
    <field name="win.system.eventID">^22$</field>
    <description>Sysmon - DNS query detected (Event ID 22)</description>
    <options>no_full_log</options>
    <group>sysmon_event_22</group>
  </rule>

  <!-- Extra (more specific) rule if needed -->
  <rule id="61650" level="5">
    <if_sid>61600</if_sid>
    <field name="win.system.providerName">Microsoft-Windows-Sysmon</field>
    <field name="win.system.eventID">22</field>
    <description>DNS query detected via Sysmon</description>
  </rule>
</group>
```

- Make sure the `<if_sid>61600</if_sid>` corresponds to the base rule for Sysmon logs (included in the Wazuh ruleset).

#### Restart Wazuh Manager

```sh
sudo systemctl restart wazuh-manager
```

#### Verify in Wazuh Dashboard

- Go to **Wazuh Web UI** -> **Security Events**.

- Search for:

```sh
rule.id:255001
```

- Or look for "DNS query detected via Sysmon".

#### Tips

- Use `ossec-logtest` to test if logs match the rules:

```sh
/var/ossec/bin/ossec-logtest
```
