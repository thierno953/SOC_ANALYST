# ELK Stack Installation and Configuration

### Update & Install Java

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-11-jdk -y
```

### Add the Elastic APT Repository

```sh
sudo mkdir -p /etc/apt/keyrings
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/keyrings/elastic.gpg

echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
```

### Install Elasticsearch

```sh
sudo apt install elasticsearch -y
sudo systemctl enable --now elasticsearch
sudo systemctl status elasticsearch
```

### Configure Elasticsearch

- Edit `/etc/elasticsearch/elasticsearch.yml`:

```sh
sudo nano /etc/elasticsearch/elasticsearch.yml
```

```sh
network.host: 0.0.0.0
http.port: 9200
```

### Restart Elasticsearch:

```sh
sudo systemctl restart elasticsearch
```

### Install Logstash

```sh
sudo apt install logstash -y
sudo systemctl enable --now logstash
sudo systemctl status logstash
```

### Install Kibana

```sh
sudo apt install kibana -y
sudo systemctl enable --now kibana
sudo systemctl status kibana
```

### Configure Kibana

- Edit `/etc/kibana/kibana.yml`:

```sh
sudo nano /etc/kibana/kibana.yml
```

- Add or modify:

```sh
server.port: 5601
server.host: "0.0.0.0"
```

### Configure Encryption Keys

```sh
cd /usr/share/kibana/bin/
sudo ./kibana-encryption-keys generate

./kibana-keystore add xpack.security.encryptionKey
./kibana-keystore add xpack.reporting.encryptionKey
./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
```

### Restart Kibana:

```sh
sudo systemctl restart kibana
```

### Allow Required Ports (UFW)

```sh
sudo ufw enable
sudo ufw allow 9200/tcp   # Elasticsearch
sudo ufw allow 5601/tcp   # Kibana
sudo ufw reload
```

- Restart Kibana:

```sh
sudo systemctl restart kibana
sudo systemctl status kibana
```

### Reset Passwords

```sh
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

### Enrollment Token & Verification Code for Kibana

```sh
cd /usr/share/elasticsearch/bin/
./elasticsearch-create-enrollment-token --scope kibana

cd /usr/share/kibana/bin/
./kibana-verification-code
```

### Secure the ELK Stack (Optional but Recommended)

- Since the setup is accessible over the internet:

  - **Enable Basic Authentication** for Kibana (built-in with Elastic Stack).

  - **Restrict access** using iptables or UFW to allow only trusted IPs.

```sh
sudo ufw allow from <your-desktop-ip> to any port 5601
sudo ufw allow from <your-desktop-ip> to any port 9200
sudo ufw reload
```

- **Optional**: Set up **Nginx** as a reverse proxy with SSL in front of Kibana for HTTPS and additional security.

### Verify Remote Access

- **Kibana**: Open browser -> http://<your-desktop-ip>:5601

- **Elasticsearch**: Test API -> https://<your-desktop-ip>:9200

## Fleet Server Configuration

- Open Fleet ports:

```sh
sudo ufw allow 8220/tcp   # Fleet Server port
sudo ufw allow 9200/tcp   # Elasticsearch port
sudo ufw reload
```

- Create Fleet Server Policy in Kibana:

  `Management > Fleet > Add Fleet Server > Generate Fleet Server policy`

- Add Fleet Server

```sh
sudo ./elastic-agent install \
  --fleet-server-es=https://<ELASTICSEARCH_IP>:9200 \
  --fleet-server-service-token=<SERVICE_TOKEN> \
  --fleet-server-policy=<POLICY_NAME> \
  --url=https://<FLEET_SERVER_IP>:8220
```

- Add Agent

`Management > Fleet > Agents > Add agent`

```sh
sudo ./elastic-agent install \
  --url=https://<FLEET_SERVER_IP>:8220 \
  --enrollment-token=<YOUR_ENROLLMENT_TOKEN>
```
