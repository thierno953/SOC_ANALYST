# Wazuh - Audit Linux Commands Run by Users (e.g., root) using auditd

- Reference:
  - [https://documentation.wazuh.com/current/proof-of-concept-guide/audit-commands-run-by-user.html](https://documentation.wazuh.com/current/proof-of-concept-guide/audit-commands-run-by-user.html)
  - [https://github.com/Neo23x0/auditd/blob/master/audit.rules](https://github.com/Neo23x0/auditd/blob/master/audit.rules)

#### Install and Start `auditd`

```sh
sudo apt update && sudo apt install auditd -y
sudo systemctl enable --now auditd
```

#### Monitor Audit Log in Real Time

```sh
tail -f /var/log/audit/audit.log
```

#### Add Audit Rules to Track Commands Run as root

- Edit audit rules file:

```sh
nano /etc/audit/rules.d/audit.rules
```

- Add the following lines to track execve syscalls by root (euid=0):

```sh
-a always,exit -F arch=b64 -F euid=0 -S execve -k audit-wazuh-c
-a always,exit -F arch=b32 -F euid=0 -S execve -k audit-wazuh-c
```

- Then apply the rules:

```sh
sudo augenrules --load
# or:
sudo auditctl -R /etc/audit/rules.d/audit.rules
```

#### Configure Wazuh Agent to Read Audit Logs

- Edit the Wazuh agent config:

```sh
nano /var/ossec/etc/ossec.conf
```

- Add inside `<ossec_config>`:

```sh
<localfile>
  <log_format>audit</log_format>
  <location>/var/log/audit/audit.log</location>
</localfile>
```

#### Create Custom Wazuh Rules

```sh
sudo nano /var/ossec/etc/rules/local_rules.xml
```

```sh
<ruleset>
  <!-- Rule 1: SSH auth failed from a specific IP -->
  <rule id="100001" level="5">
    <if_sid>5716</if_sid>
    <srcip>1.1.1.1</srcip>
    <description>sshd: authentication failed from IP 1.1.1.1</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5</group>
  </rule>

  <!-- Rule 2: Command executed as root -->
  <rule id="100014" level="12">
    <if_sid>80700</if_sid>
    <field name="audit.type">SYSCALL</field>
    <field name="audit.euid">0</field>
    <description>Audit: Command executed by root (euid=0)</description>
    <group>audit_command</group>
  </rule>

  <!-- Rule 3: SSH brute force attempt -->
  <rule id="100100" level="10">
    <if_sid>5710</if_sid>
    <match>authentication failure</match>
    <description>SSH Brute Force Attack Detected</description>
    <group>ssh,authentication_failed</group>
  </rule>
</ruleset>
```

### Restart Services

```sh
sudo systemctl restart wazuh-manager
sudo systemctl restart auditd
sudo systemctl restart wazuh-agent
```

### Test Tracking

- As root, run:

```sh
lsmod
id
useradd testuser
```

- Then check logs:

```sh
grep testuser /var/log/audit/audit.log
```

### Use Wazuh Dashboard

```sh
Security Events -> Filters -> rule.id:100014
```

![WAZUH](/Wazuh/assets/14.png)

![WAZUH](/Wazuh/assets/15.png)
