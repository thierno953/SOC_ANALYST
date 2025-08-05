root@siem:/home/siem# df -h /
root@siem:/home/siem# sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
root@siem:/home/siem# sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
root@siem:/home/siem# df -h /
--------------------------------
root@iris-shuffle:/opt/Shuffle# nano docker-compose.yml
```
- "OPENSEARCH_JAVA_OPTS=-Xms1024m -Xmx1024m"

ou
- "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"

ou
- "OPENSEARCH_JAVA_OPTS=-Xms2g -Xmx2g"
```
root@iris-shuffle:/opt/Shuffle# sudo fallocate -l 2G /swapfile
root@iris-shuffle:/opt/Shuffle# sudo chmod 600 /swapfile
root@iris-shuffle:/opt/Shuffle# sudo mkswap /swapfile
root@iris-shuffle:/opt/Shuffle# sudo swapon /swapfile
root@iris-shuffle:/opt/Shuffle# swapon --show
root@iris-shuffle:/opt/Shuffle# /swapfile none swap sw 0 0
------------------------------------
root@siem:/home/siem# sudo nano /var/ossec/etc/internal_options.conf
```sh
vulnerability-detection.disable_scan_manager=1 --> vulnerability-detection.disable_scan_manager=0
```
root@siem:/home/siem# sudo systemctl restart wazuh-manager
----------------------------------
root@iris-shuffle:/opt/iris-web# docker compose logs app | grep 'admin'
iriswebapp_app  | 2025-07-24 08:58:37 :: INFO :: post_init :: run_post_init :: Creating first administrative user
iriswebapp_app  | 2025-07-24 08:58:37 :: INFO :: post_init :: create_safe_admin :: Creating first admin user with username "administrator"
iriswebapp_app  | 2025-07-24 08:58:37 :: WARNING :: post_init :: create_safe_admin :: >>> Administrator password: 6V7I'?G_X&?rMz@%
iriswebapp_app  | 2025-07-24 08:58:40 :: INFO :: post_init :: run_post_init :: You can now login with user administrator and password >>> 6V7I'?G_X&?rMz@% <<< on 443
root@iris-shuffle:/opt/iris-web#
------------------------------------------------

sudo nano /var/ossec/etc/ossec.conf

```sh
<ossec_config>
  ...
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/TON/WEBHOOK/ICI</hook_url>
    <level>5</level>
    <alert_format>json</alert_format>
    <json_output>
      {
        "text": "*Nouvelle alerte Wazuh* :\n*Agent:* _$(agent.name)_\n*Type:* $(rule.level) - $(rule.description)\n*Date:* $(timestamp)\n*Détails:* $(full_log)"
      }
    </json_output>
  </integration>
  ...
</ossec_config>
```

sudo chmod 600 /var/ossec/etc/ossec.conf
sudo chown root:ossec /var/ossec/etc/ossec.conf

sudo systemctl restart wazuh-manager
# ou
sudo service wazuh-manager restart


sudo /var/ossec/bin/manager_control -l 10 -m "Test Slack Alert from Wazuh"


```sh
<ossec_config>
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/XXXXX/YYYYY/ZZZZZ</hook_url>
    <level>10</level>
    <alert_format>json</alert_format>
  </integration>

  <integration>
    <name>email</name>
    <email_from>wazuh@example.com</email_from>
    <email_to>admin@example.com</email_to>
    <smtp_server>smtp.example.com</smtp_server>
    <level>10</level>
  </integration>
</ossec_config>
```

```sh
<group name="command_detection">
  <rule id="100001" level="10">
    <if_sid>5710</if_sid>
    <match>nmap|netcat|nc|hydra</match>
    <description>Commandes de scan détectées</description>
  </rule>
</group>

```
==================================================

# https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html

apt-get install curl -y
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get update
WAZUH_MANAGER="IP_WAZUH_SERVER" apt-get install wazuh-agent
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
apt-get update

# https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html



ufw allow from 192.168.129.246
ufw allow from 192.168.129.245
ufw allow from 192.168.129.228


# https://documentation.wazuh.com/current/user-manual/manager/event-logging.html#archiving-event-logs

root@master01:~# nano /var/ossec/etc/ossec.conf

```sh
<ossec_config>
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
</ossec_config>
```

root@master01:~# systemctl restart wazuh-manager
root@wazuh:/# nano /var/ossec/logs/archives/archives.json
root@wazuh:/# nano /etc/filebeat/filebeat.yml

```sh
archives:
 enabled: true
```

root@wazuh:/# service filebeat restart

# https://documentation.wazuh.com/current/user-manual/reference/centralized-configuration.html#centralized-configuration-process
# https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/configuration.html#windows-event-channel

root@master01:~# nano /var/ossec/etc/shared/default/agent.conf

```sh
<agent_config>

  <!-- Shared agent configuration here -->
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
</agent_config>
```

root@master01:~# /var/ossec/bin/verify-agent-conf
root@master01:~# systemctl restart wazuh-manager
root@master01:~# /var/ossec/bin/agent_groups -S -i 001
root@master01:~# /var/ossec/bin/agent_groups -S -i 002
root@master01:~# /var/ossec/bin/agent_groups -S -i 003
root@master01:~# /var/ossec/bin/agent_groups -l

```sh
data.win.system.channel:*Sysmon* 
data.win.system.channel:*Sysmon* and agent.name: Thierno
```

#  Accessing Linux Default External Collected Logs using Index Patterns and Dashboards

root@master01:~# nano /var/ossec/etc/ossec.conf

```sh

```


# https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html



PS C:\Users\thier> Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.12.0-1.msi" -OutFile "$env:TEMP\wazuh-agent.msi"
PS C:\Users\thier> Start-Process msiexec.exe -Wait -ArgumentList "/i `"$env:TEMP\wazuh-agent.msi`" /qn WAZUH_MANAGER='192.168.129.247' WAZUH_AGENT_GROUP='Windows' WAZUH_AGENT_NAME='Thierno'"
PS C:\Users\thier> Get-Service | Where-Object { $_.DisplayName -like "*Wazuh*" }

Status   Name               DisplayName
------   ----               -----------
Stopped  WazuhSvc           Wazuh


PS C:\Users\thier> NET START WazuhSvc

The Wazuh service was started successfully.

<integration>
    <name>shuffle</name>
    <hook_url>http://192.168.129.86:3001/api/v1/hooks/webhook_7e120033-54eb-4755-8c7a-1b94cf15aebd </hook_url>
    <rule_id>100002</rule_id>
    <alert_format>json</alert_format>
  </integration>

PS C:\Users\thier> 

------------------------

nano /var/ossec/etc/ossec.conf 

```sh
<!-- IRIS integration -->
<integration>
  <name>custom-iris.py</name>
  <hook_url>https://192.168.129.81/alerts/add</hook_url>
  <api_key>sXrxMSzeIx6UjJ9m6UVBFlFxwXod62OY08yrrU4FkWSuzB22N4t4sHaJeExdRWAf0gkRzHnltTmrchjmnQQTWQ</api_key>
  <alert_format>json</alert_format>
  <level>10</level>
</integration>
```
root@master01:~/Wazuh-IRIS-integration# nano /var/ossec/etc/rules/local_rules.xml
```sh
<rule id="100100" level="10">
  <program_name>test_wazuh</program_name>
  <description>Test alert for IRIS integration</description>
</rule>
```
root@master01:~/Wazuh-IRIS-integration# logger -t test_wazuh "This is a test alert for IRIS integration"
root@master01:~/Wazuh-IRIS-integration# tail -f /var/ossec/logs/alerts/alerts.log

================================================================================================

-------------------
**wazuh:** -hgmJmj1dha78ANXAHB-dFUrGOgNsYzZcaL_fHErKFxfOrUMcQlJxls6rP19-8wG68OY0-SfTRlQ7ssOirqx5A
--------------------

# https://wazuh.com/blog/enhancing-incident-response-with-wazuh-and-dfir-iris-integration/

root@siem:~# nano /var/ossec/integrations/custom-wazuh_iris.py

```sh
#!/var/ossec/framework/python/bin/python3
# custom-wazuh_iris.py
# Custom Wazuh integration script to send alerts to DFIR-IRIS

import sys
import json
import requests
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(filename='/var/ossec/logs/integrations.log', level=logging.INFO, 
                    format='%(asctime)s %(levelname)s: %(message)s', datefmt='%Y-%m-%d %H:%M:%S')

# Function to create a formatted string from alert details
def format_alert_details(alert_json):
    rule = alert_json.get("rule", {})
    agent = alert_json.get("agent", {})
    
    # Extracting MITRE information from the nested 'rule' structure
    mitre = rule.get("mitre", {})
    mitre_ids = ', '.join(mitre.get("id", ["N/A"]))
    mitre_tactics = ', '.join(mitre.get("tactic", ["N/A"]))
    mitre_techniques = ', '.join(mitre.get("technique", ["N/A"]))

    details = [
        f"Rule ID: {rule.get('id', 'N/A')}",
        f"Rule Level: {rule.get('level', 'N/A')}",
        f"Rule Description: {rule.get('description', 'N/A')}",
        f"Agent ID: {agent.get('id', 'N/A')}",
        f"Agent Name: {agent.get('name', 'N/A')}",
        f"MITRE IDs: {mitre_ids}",
        f"MITRE Tactics: {mitre_tactics}",
        f"MITRE Techniques: {mitre_techniques}",
        f"Location: {alert_json.get('location', 'N/A')}",
        f"Full Log: {alert_json.get('full_log', 'N/A')}"
    ]
    return '\n'.join(details)

def main():
    # Read parameters when integration is run
    if len(sys.argv) < 4:
        logging.error("Insufficient arguments provided. Exiting.")
        sys.exit(1)
    
    alert_file = sys.argv[1]
    api_key = sys.argv[2]
    hook_url = sys.argv[3]

    # Read the alert file
    try:
        with open(alert_file) as f:
            alert_json = json.load(f)
    except Exception as e:
        logging.error(f"Failed to read alert file: {e}")
        sys.exit(1)

    # Prepare alert details
    alert_details = format_alert_details(alert_json)

    # Convert Wazuh rule levels(0-15) -> IRIS severity(1-6)
    alert_level = alert_json.get("rule", {}).get("level", 0)
    if alert_level < 5:
        severity = 2
    elif alert_level >= 5 and alert_level < 7:
        severity = 3
    elif alert_level >= 7 and alert_level < 10:
        severity = 4
    elif alert_level >= 10 and alert_level < 13:
        severity = 5
    elif alert_level >= 13:
        severity = 6
    else:
        severity = 1

    # Generate request
    payload = json.dumps({
        "alert_title": alert_json.get("rule", {}).get("description", "No Description"),
        "alert_description": alert_details,
        "alert_source": "Wazuh",
        "alert_source_ref": alert_json.get("id", "Unknown ID"),
        "alert_source_link": "https://<IP ADDRESS>/app/wz-home",  # Replace with actual Wazuh dashboard IP address
        "alert_severity_id": severity,
        "alert_status_id": 2,  # 'New' status
        "alert_source_event_time": alert_json.get("timestamp", "Unknown Timestamp"),
        "alert_note": "",
        "alert_tags": f"wazuh,{alert_json.get('agent', {}).get('name', 'N/A')}",
        "alert_customer_id": 1,  # '1' for default 'IrisInitialClient'
        "alert_source_content": alert_json  # raw log
    })

    # Send request to IRIS
    try:
        response = requests.post(hook_url, data=payload, headers={"Authorization": "Bearer " + api_key, "content-type": "application/json"}, verify=False)
        if response.status_code in [200, 201, 202, 204]:
            logging.info(f"Sent alert to IRIS. Response status code: {response.status_code}")
        else:
            logging.error(f"Failed to send alert to IRIS. Response status code: {response.status_code}")
    except Exception as e:
        logging.error(f"Failed to send alert to IRIS: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

root@siem:~# chmod 750 /var/ossec/integrations/custom-wazuh_iris.py
root@siem:~# chown root:wazuh /var/ossec/integrations/custom-wazuh_iris.py
root@siem:~# nano /var/ossec/etc/ossec.conf

```sh
# custom integration
<!-- IRIS integration -->
<integration>
  <name>custom-wazuh_iris.py</name>
  <hook_url>https://192.168.129.91/alerts/add</hook_url>
  <level>7</level>
  <api_key>-hgmJmj1dha78ANXAHB-dFUrGOgNsYzZcaL_fHErKFxfOrUMcQlJxls6rP19-8wG68OY0-SfTRlQ7ssOirqx5A</api_key>                                       
  <alert_format>json</alert_format>
</integration>
```

root@siem:~# systemctl restart wazuh-manager
root@siem:~# systemctl status wazuh-manager
root@siem:/var/ossec/logs# nano /var/ossec/etc/rules/local_rules.xml

```sh
<group name="local,syslog,sshd">
  <rule id="100001" level="5">
    <if_sid>5716</if_sid>
    <srcip>1.1.1.1</srcip>
    <description>sshd: authentication failed from IP 1.1.1.1.</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5</group>
  </rule>
</group>
```

```sh
root@siem:/var/ossec/logs# nano /var/ossec/logs/alert_test_iris.json
{
  "alert_title": "Example test alert",
  "alert_severity_id": 3,
  "alert_status_id": 1,
  "alert_customer_id": 2,
  "alert_description": "This is a test full log entry",
  "alert_source": "Wazuh agent-test",
  "alert_extra": {
    "timestamp": "2025-07-24T10:00:00Z",
    "rule_id": "100001",
    "mitre": {
      "id": ["T1003"],
      "tactic": ["Credential Access"],
      "technique": ["OS Credential Dumping"]
    },
    "agent_id": "001",
    "location": "Test Location"
  }
}
root@siem:/var/ossec/logs# curl -k -X POST https://192.168.129.91/alerts/add \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer -hgmJmj1dha78ANXAHB-dFUrGOgNsYzZcaL_fHErKFxfOrUMcQlJxls6rP19-8wG68OY0-SfTRlQ7ssOirqx5A" \
  -d @/var/ossec/logs/alert_test_iris.json
```

root@siem:/var/ossec/logs# sudo systemctl restart wazuh-manager
root@siem:/var/ossec/logs# tail -f integrations.log
root@siem:/var/ossec/logs# tail -f ossec.log 
root@siem:/var/ossec/logs# grep ERROR ossec.log 

-----------------------

root@attack:/home/attack# hydra -t 1 -V -f -l baben -P password.txt 192.168.129.90 ssh

--------------------------

root@siem:/home/siem# nano /var/ossec/etc/ossec.conf


<integration>
  <name>custom-iris.py</name>
  <hook_url>http://192.168.129.91/alerts/add</hook_url>
  <level>7</level>
  <api_key>-hgmJmj1dha78ANXAHB-dFUrGOgNsYzZcaL_fHErKFxfOrUMcQlJxls6rP19-8wG68OY0-SfTRlQ7ssOirqx5A</api_key>
  <alert_format>json</alert_format>
</integration>

<!-- ... Rest of config ... -->

<integration>
  <name>shuffle</name>
  <hook_url>https://192.168.129.91:3443/api/v1/hooks/webhook_ed2ff268-2a88-4cf5-abf2-811668791117</hook_url>
  <level>3</level>
  <alert_format>json</alert_format>
</integration>