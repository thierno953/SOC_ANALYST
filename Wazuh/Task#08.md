# Wazuh - Audit Linux Commands Run by Users (e.g., root) using auditd

- Reference:
  - [https://documentation.wazuh.com/current/proof-of-concept-guide/audit-commands-run-by-user.html](https://documentation.wazuh.com/current/proof-of-concept-guide/audit-commands-run-by-user.html)
  - [https://github.com/Neo23x0/auditd/blob/master/audit.rules](https://github.com/Neo23x0/auditd/blob/master/audit.rules)

#### Install and Start `auditd` (Audit Daemon)

```sh
apt install auditd -y && systemctl enable --now auditd
```

#### Monitor Audit Log in Real Time (Optional)

```sh
tail -f /var/log/audit/audit.log
```

#### Add Audit Rules to Track Command Executions by Root

- Edit audit rules file:

```sh
nano /etc/audit/rules.d/soc.rules
```

- Add the following lines to track execve syscalls by root (euid=0):

```sh
[https://github.com/Neo23x0/auditd/blob/master/audit.rules](https://github.com/Neo23x0/auditd/blob/master/audit.rules)
```

- Then apply the rules:

```sh
auditctl -R /etc/audit/audit.rules
```

#### Configure Wazuh Agent to Read Audit Logs

- Edit the Wazuh agent config:

```sh
nano /var/ossec/etc/ossec.conf
```

- Add this inside `<ossec_config>`:

```sh
<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

#### Restart Wazuh Agent and Auditd

```sh
systemctl restart wazuh-agent auditd
```

#### Test Auditd Tracking

- Try a few commands as root to generate audit logs:

```sh
lsmod
modinfo intel_rapl_common
useradd malicious
netstat
```

- Check if they're logged:

```sh
cat /var/log/audit/audit.log | grep malicious
```

#### Create Custom Wazuh Rules

- Create local rules directory if it doesn't exist:

```sh
mkdir -p /var/ossec/etc/rules
```

- Edit or create the local rules file:

```sh
nano /var/ossec/etc/rules/local_rules.xml
```

- Paste this custom ruleset:

```sh
<ruleset>
  <group name="audit">
    <rule id="100014" level="12">
      <if_sid>80700</if_sid>
      <field name="audit.type">SYSCALL</field>
      <field name="audit.euid">0</field>
      <description>Audit: Command executed by root (euid=0)</description>
      <group>audit_command</group>
    </rule>
  </group>

  <group name="ssh,authentication_failed">
    <rule id="100100" level="10">
      <if_sid>5710</if_sid>
      <match>authentication failure</match>
      <description>SSH Brute Force Attack Detected</description>
    </rule>
  </group>
</ruleset>
```

- Restart the Wazuh agent to apply changes:

```sh
systemctl restart wazuh-agent
```

#### Validation

- Use `/var/ossec/bin/ossec-logtest` to test if rules are triggered

- Or open the **Wazuh Dashboard** -> **Security Events** and filter by:

```sh
rule.id: 100014
```
