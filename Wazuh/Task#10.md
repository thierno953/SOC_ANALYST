# DFIR-IRIS Installation and Integration with Wazuh

## Docker Installation

```sh
# Update your system
sudo apt-get update

# Install required packages
sudo apt-get install ca-certificates curl gnupg -y

# Create keyrings directory for Docker
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set permissions
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add Docker repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update apt package index
sudo apt-get update

# Install Docker components
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add your user to the Docker group (to run Docker without sudo)
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker
```

#### DFIR-IRIS Deployment (Docker-based)

- Official documentation: [https://docs.dfir-iris.org/getting_started/](https://docs.dfir-iris.org/getting_started/)

#### Wazuh & IRIS Integration

- GitHub Repository: [https://github.com/nateuribe/Wazuh-IRIS-integration](https://github.com/nateuribe/Wazuh-IRIS-integration)

#### Configure Wazuh Manager

- Edit the `ossec.conf` file:

```sh
sudo nano /var/ossec/etc/ossec.conf
```

#### Add the following integration block:

```sh
<integration>
  <name>custom-wazuh_iris.py</name>
  <hook_url>https://<IRIS_IP_ADDRESS>/alerts/add</hook_url>
  <level>7</level>
  <api_key><IRIS_API_KEY></api_key>
  <alert_format>json</alert_format>
</integration>
```

#### Restart Wazuh Manager:

```sh
sudo systemctl restart wazuh-manager
```

## Configure Log Monitoring

- Add these localfile entries to `/var/ossec/etc/ossec.conf` to monitor authentication and system logs:

```sh
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>
```

## Add Custom Rules

- Edit your local rules file:

```sh
sudo nano /var/ossec/etc/rules/local_rules.xml
```

```sh
<group name="local">
  <rule id="150001" level="7">
    <match>Wazuh test alert level 7</match>
    <description>Test alert level 7 triggered</description>
  </rule>
  <rule id="150002" level="3">
    <match>Wazuh test alert level 3</match>
    <description>Test alert level 3 triggered</description>
  </rule>
</group>
```

## Validate and Restart Wazuh

```sh
/var/ossec/bin/wazuh-analysisd -t
sudo systemctl restart wazuh-manager
```

## Testing Alerts

```sh
logger -p auth.alert "Wazuh test alert level 3"
logger -p user.alert "Wazuh test alert level 7"
```

## Check Wazuh alerts by running

```sh
sudo cat /var/ossec/logs/alerts/alerts.json | grep "Wazuh test alert"
sudo tail -f /var/ossec/logs/alerts/alerts.json
```

![WAZUH](/Wazuh/assets/19.png)

![WAZUH](/Wazuh/assets/20.png)

![WAZUH](/Wazuh/assets/21.png)

![WAZUH](/Wazuh/assets/22.png)
