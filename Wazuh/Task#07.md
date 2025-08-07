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

#### Simulate SQL Injection Attacks

- Use `curl` or `sqlmap` from another machine or terminal:

```sh
curl "http://<WAZUH_AGENT_IP>/users/?id=1 OR 1=1"
curl "http://<WAZUH_AGENT_IP>/users/?id=SELECT+*+FROM+users"
sqlmap -u "http://<WAZUH_AGENT_IP>/users/?id=1"
curl "http://<WAZUH_AGENT_IP>/?id=1' OR '1'='1"
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
    <match>select.*from</match>
    <description>Custom Rule - SQL Injection detected</description>
    <group>web, sqli</group>
  </rule>

  <rule id="100101" level="10">
    <match>(&lt;script|javascript:|onerror=|onload=)</match>
    <description>Custom Rule - XSS attack attempt detected</description>
    <group>web, xss</group>
  </rule>

  <rule id="100102" level="10">
    <match>(\.\./|\.\.%2f|/etc/passwd|/proc/self/environ)</match>
    <description>Custom Rule - LFI attack attempt detected</description>
    <group>web, lfi</group>
  </rule>
</group>
```

- Then restart the Wazuh manager:

```sh
sudo systemctl restart wazuh-manager
```

![WAZUH](/Wazuh/assets/13.png)
