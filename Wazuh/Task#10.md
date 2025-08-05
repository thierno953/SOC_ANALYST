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
<ossec_config>
  <!-- IRIS Integration -->
  <integration>
    <name>custom-wazuh_iris.py</name>
    <hook_url>https://<IRIS_IP_ADDRESS>/alerts/add</hook_url>
    <level>7</level>
    <api_key><IRIS_API_KEY></api_key> <!-- Replace with your IRIS API key -->
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

- Replace `<IRIS_IP_ADDRESS>` and `<IRIS_API_KEY>` with your actual values.

#### Restart Wazuh Manager:

```sh
sudo systemctl restart wazuh-manager
```
