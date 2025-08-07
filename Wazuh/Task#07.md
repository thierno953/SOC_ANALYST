# Wazuh – Detecting SQL Injection Attacks via Apache Logs

- Reference: [https://documentation.wazuh.com/current/proof-of-concept-guide/detect-web-attack-sql-injection.html](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-web-attack-sql-injection.html)

#### Install Apache2 on the Agent

```sh
sudo apt update && sudo apt install apache2
```

#### Configure Wazuh Agent to Monitor Apache Access Logs

- Edit the Wazuh agent configuration:

```sh
sudo nano /var/ossec/etc/ossec.conf
```

- Add this block inside `<ossec_config>`

```sh
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```

#### Restart the Wazuh Agent

```sh
sudo systemctl restart wazuh-agent
```

#### Create a Custom Wazuh Rule for SQLi

- Edit the local rules file:

```sh
nano /var/ossec/ruleset/rules/0999-local_rules.xml
```

- Paste this rule:

```sh
<group name="apache_custom, web, attack">
  <rule id="100100" level="10">
    <match>select</match>
    <description>Custom Rule - SQL Injection detected (select)</description>
    <group>web, sqli</group>
  </rule>
  <rule id="100101" level="10">
    <match>from</match>
    <description>Custom Rule - SQL Injection detected (from)</description>
    <group>web, sqli</group>
  </rule>

  <rule id="100110" level="10">
    <match>&lt;script</match>
    <description>Custom Rule - XSS attack attempt detected (&lt;script)</description>
    <group>web, xss</group>
  </rule>
  <rule id="100111" level="10">
    <match>javascript:</match>
    <description>Custom Rule - XSS attack attempt detected (javascript:)</description>
    <group>web, xss</group>
  </rule>

  <rule id="100120" level="10">
    <match>../</match>
    <description>Custom Rule - LFI attack attempt detected (../)</description>
    <group>web, lfi</group>
  </rule>
  <rule id="100121" level="10">
    <match>/etc/passwd</match>
    <description>Custom Rule - LFI attack attempt detected (/etc/passwd)</description>
    <group>web, lfi</group>
  </rule>
</group>
```

- Then restart the Wazuh manager:

```sh
sudo systemctl restart wazuh-manager
```

#### Simulate SQL Injection Attacks

```sh
curl "http://<WAZUH_AGENT_IP>/users/?id=SELECT+*+FROM+users"
curl "http://<WAZUH_AGENT_IP>/?id=1' OR '1'='1"
sqlmap -u "http://<WAZUH_AGENT_IP>/users/?id=1"
```

![WAZUH](/Wazuh/assets/13.png)
