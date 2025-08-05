# ELK Stack (Elasticsearch, Logstash, Kibana) with Security Configuration and Fleet Server Setup

- Update & Install Java

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-11-jdk -y
```

- Add the Elastic APT Repository

```sh
sudo mkdir -p /etc/apt/keyrings
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/keyrings/elastic.gpg

echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
```

- Install Elasticsearch

```sh
sudo apt install elasticsearch -y
sudo systemctl enable --now elasticsearch
sudo systemctl status elasticsearch
```

- Configure Elasticsearch

```sh
sudo nano /etc/elasticsearch/elasticsearch.yml
```

- Add or modify the following lines:

```sh
network.host: 0.0.0.0
http.port: 9200
```

- Then restart:

```sh
sudo systemctl restart elasticsearch
sudo systemctl status elasticsearch
```

- Install Logstash

```sh
sudo apt install logstash -y
sudo systemctl enable --now logstash
sudo systemctl status logstash
```

- Install Kibana

```sh
sudo apt install kibana -y
sudo systemctl enable --now kibana
sudo systemctl status kibana
```

- Configure Kibana

```sh
sudo nano /etc/kibana/kibana.yml
```

- Add or modify:

```sh
server.port: 5601
server.host: "0.0.0.0"
```

- Restart:

```sh
sudo systemctl restart kibana
sudo systemctl status kibana
```

- Configure Encryption Keys for Kibana
  - Generate keys:

```sh
cd /usr/share/kibana/bin/
sudo ./kibana-encryption-keys generate
```

- Add them to the keystore:

```sh
./kibana-keystore add xpack.security.encryptionKey
./kibana-keystore add xpack.reporting.encryptionKey
./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
```

- Restart Kibana:

```sh
sudo systemctl restart kibana
sudo systemctl status kibana
```

- Allow Required Ports (UFW)

```sh
sudo ufw enable
sudo ufw allow 9200/tcp
sudo ufw allow 5601/tcp
sudo ufw reload
```

- Reset Passwords

```sh
# For kibana_system user:
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

# For elastic user:
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

- Enrollment Token & Verification Code for Kibana

```sh
cd /usr/share/elasticsearch/bin/
./elasticsearch-create-enrollment-token --scope kibana

cd /usr/share/kibana/bin/
./kibana-verification-code
```

![ELK](/Elastic_Stack_Ubuntu/assets/01.png)

## Fleet Server Configuration

- On the Fleet Agent machine

```sh
sudo ufw enable
sudo ufw allow 8220/tcp
sudo ufw allow 9200/tcp
sudo ufw reload
```

- In Kibana `Management > Fleet > Add Fleet Server > Generate Fleet Server policy`

- On the Agent (Run the command provided by Kibana)

```sh
sudo ./elastic-agent install \
  --url=https://<FLEET_SERVER_IP>:8220 \
  --enrollment-token=<your_fleet_enrollment_token>
```

![ELK](/Elastic_Stack_Ubuntu/assets/02.png)
