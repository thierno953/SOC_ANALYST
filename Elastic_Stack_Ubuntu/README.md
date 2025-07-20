# ELK Stack (Elasticsearch, Logstash, Kibana) avec configuration de sécurité et du Fleet Server

- Mise à jour et installation de Java

```sh
sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-11-jdk -y
```

- Ajout du dépôt Elastic

```sh
sudo mkdir -p /etc/apt/keyrings
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/keyrings/elastic.gpg

echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
```

- Installation d’Elasticsearch

```sh
sudo apt install elasticsearch -y
sudo systemctl enable --now elasticsearch
sudo systemctl status elasticsearch
```

- Configuration d'Elasticsearch

```sh
sudo nano /etc/elasticsearch/elasticsearch.yml
```

- Modifie ou ajoute :

```sh
network.host: 0.0.0.0
http.port: 9200
```

```sh
sudo systemctl restart elasticsearch
sudo systemctl status elasticsearch
```

- Installation de Logstash

```sh
sudo apt install logstash -y
sudo systemctl enable --now logstash
sudo systemctl status logstash
```

- Installation de Kibana

```sh
sudo apt install kibana -y
sudo systemctl enable --now kibana
sudo systemctl status kibana
```

- Configuration de Kibana

```sh
sudo nano /etc/kibana/kibana.yml
```

- Ajoute/modifie :

```sh
server.port: 5601
server.host: "0.0.0.0"
```

```sh
sudo systemctl restart kibana
sudo systemctl status kibana
```

- Configuration des clés de chiffrement dans Kibana

```sh
cd /usr/share/kibana/bin/
sudo ./kibana-encryption-keys generate
```

- Ajoute les clés générées dans le keystore :

```sh
./kibana-keystore add xpack.security.encryptionKey
./kibana-keystore add xpack.reporting.encryptionKey
./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
```

```sh
sudo systemctl restart kibana
sudo systemctl status kibana
```

- Pare-feu (UFW)

```sh
sudo ufw enable
sudo ufw allow 9200/tcp
sudo ufw allow 5601/tcp
sudo ufw reload
```

- Réinitialisation des mots de passe

```sh
# Pour l'utilisateur kibana_system :
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

# Pour l'utilisateur elastic :
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

- Jeton d’enrôlement & vérification Kibana

```sh
cd /usr/share/elasticsearch/bin/
./elasticsearch-create-enrollment-token --scope kibana

cd /usr/share/kibana/bin/
./kibana-verification-code
```

![ELK](/Elastic_Stack_Ubuntu/assets/01.png)

#### Configuration du Fleet Server

- Sur la machine fleet-agent

```sh
sudo ufw enable
sudo ufw allow 8220/tcp
sudo ufw allow 9200/tcp
sudo ufw reload
```

- Sur Kibana :

`Management > Fleet > Add Fleet Server > Generate Fleet Server policy`

- Ensuite, sur l'agent :

```sh
sudo ./elastic-agent install \
  --url=https://<IP_FLEET_SERVER>:8220 \
  --enrollment-token=<token>
```

![ELK](/Elastic_Stack_Ubuntu/assets/02.png)

