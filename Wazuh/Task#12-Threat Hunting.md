# Threat Hunting with Wazuh

### Create a malware hash list

```sh
nano /var/ossec/etc/lists/malware-hashes
```

- Add known malware hashes:

```sh
e0ec2cd43f71c80d42cd7b0f17802c73:mirai
55142f1d393c5ba7405239f232a6c059:xbash
```

### Include the list in `ossec.conf`

```sh
nano /var/ossec/etc/ossec.conf
```

- Add inside the `<ruleset>` block:

```sh
<ruleset>
   <list>etc/lists/malware-hashes</list>
</ruleset>
```

### Create a custom detection rule

```sh
nano /var/ossec/etc/rules/local_rules.xml
```

- Insert the following rule:

```sh
<group name="malware,">
  <rule id="110002" level="13">
    <!-- The if_sid tag references the built-in FIM rules (550 = file added, 554 = file modified) -->
    <if_sid>550, 554</if_sid>
    <list field="md5" lookup="match_key">etc/lists/malware-hashes</list>
    <description>File with known malware hash detected: $(file)</description>
    <mitre>
      <id>T1204.002</id> <!-- User Execution: Malicious File -->
    </mitre>
  </rule>
</group>
```

### Restart Wazuh Manager

```sh
systemctl restart wazuh-manager
```

### Configure Windows Agent for FIM monitoring

- Add the following directory to monitor in `ossec.conf` on the **Windows agent**:

```sh
<directories recursion_level="1" realtime="yes" check_md5="yes">C:\Users\win10\Desktop\wazuh</directories>
```

### Test with a sample file on Windows

```sh
echo "file test mirai" > C:\Users\win10\Desktop\wazuh\test.exe
Get-FileHash -Algorithm MD5 "C:\Users\win10\Desktop\wazuh\test.exe"
```

### Fix permissions on the hash list (Linux manager)

```sh
chown wazuh:wazuh /var/ossec/etc/lists/malware-hashes
```
