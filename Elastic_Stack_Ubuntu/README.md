- Installing ELK Stack

```sh
root@elastic:~# sudo apt update && sudo apt upgrade -y
root@elastic:~# sudo apt install openjdk-11-jdk -y
root@elastic:~# wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
root@elastic:~# sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" > /etc/apt/sources.list.d/elastic-8.x.list'
```

```sh
root@elastic:~# sudo apt install elasticsearch -y
root@elastic:~# sudo systemctl enable --now elasticsearch
root@elastic:~# sudo systemctl start elasticsearch
root@elastic:~# sudo systemctl status elasticsearch
```

- Install Logstash

```sh
root@elastic:~# sudo apt install logstash -y
root@elastic:~# sudo systemctl enable --now logstash
root@elastic:~# sudo systemctl start logstash
root@elastic:~# sudo systemctl status logstash
```

- Install Kibana

```sh
root@elastic:~# sudo apt install kibana -y
root@elastic:~# sudo systemctl enable --now kibana
root@elastic:~# sudo systemctl start kibana
root@elastic:~# sudo systemctl status kibana
```

```sh
root@elastic:~# sudo nano /etc/elasticsearch/elasticsearch.yml
```

```sh
#network.host: ["192.168.1.10", "127.0.0.1"]
network.host: 0.0.0.0
http.port: 9200

xpack.security.enabled: true
xpack.security.http.ssl.enabled: true
```

```sh
root@elastic:~# sudo systemctl restart elasticsearch
root@elastic:~# sudo systemctl status elasticsearch
```

```sh
root@elastic:~# sudo nano /etc/kibana/kibana.yml
```

```sh
server.port: 5601
server.host: "0.0.0.0"
```

```sh
root@elastic:~# sudo systemctl restart kibana
root@elastic:~# sudo systemctl status kibana
```

```sh
root@elastic:~# cd /usr/share/kibana/bin/
```

```sh
root@elastic:/usr/share/kibana/bin# sudo ./kibana-encryption-keys generate

xpack.encryptedSavedObjects.encryptionKey: 08447a94ef3c436bd5f79cd51aa562f9
xpack.reporting.encryptionKey: a155a0704b24eb0f408d677bc8026712
xpack.security.encryptionKey: 02973590368e907437fc716b8bcdea8f
```

```sh
root@elastic:/usr/share/kibana/bin# ./kibana-keystore add xpack.security.encryptionKey
Enter value for xpack.security.encryptionKey: ********************************
root@elastic:/usr/share/kibana/bin#
root@elastic:/usr/share/kibana/bin# ./kibana-keystore add xpack.reporting.encryptionKey
Enter value for xpack.reporting.encryptionKey: ********************************
root@elastic:/usr/share/kibana/bin#
root@elastic:/usr/share/kibana/bin# ./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
Enter value for xpack.encryptedSavedObjects.encryptionKey: ********************************
root@elastic:/usr/share/kibana/bin#
```

```sh
root@elastic:/usr/share/kibana/bin# sudo systemctl restart kibana
root@elastic:/usr/share/kibana/bin# sudo systemctl status kibana
```

```sh
root@elastic:/usr/share/kibana/bin# sudo ufw enable
root@elastic:/usr/share/kibana/bin# sudo ufw allow 9200/tcp
root@elastic:/usr/share/kibana/bin# sudo ufw allow 5601/tcp
root@elastic:/usr/share/kibana/bin# sudo ufw reload
```

```sh
root@elastic:/usr/share/kibana/bin# sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

# https://<IP>:9200/

Password for the [kibana_system] user successfully reset.
New value: 3-J61NXIFoZMJnbF+EAZ
```

```sh
root@elastic:/usr/share/kibana/bin# sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
 
# http://<IP>:5601/

Password for the [elastic] user successfully reset.
New value: ojHdKyVvsaSMkxo8GQSw
```

```sh
root@elastic:/usr/share/kibana/bin# cd /usr/share/elasticsearch/bin/
root@elastic:/usr/share/elasticsearch/bin# ./elasticsearch-create-enrollment-token --scope kibana
root@elastic:/usr/share/elasticsearch/bin# cd /usr/share/kibana/bin/
root@elastic:/usr/share/kibana/bin# ./kibana-verification-code 
```

#### Lab Set up: Fleet Server Setup

```sh
root@fleet-agent:~# sudo ufw enable
root@fleet-agent:~# sudo ufw allow 8220/tcp
root@fleet-agent:~# sudo ufw allow 9200/tcp
root@fleet-agent:~# sudo ufw reload
```

`Management > Fleet > Add Fleet Server > Generate Fleet Server policy`

```sh
sudo ./elastic-agent install \
  --url=https://<IP_FLEET_SERVER>:8220 \
  --enrollment-token=<token>
```
