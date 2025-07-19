# Investigation de sécurité d'une machine Ubuntu avec Splunk

- [Splunk Enterprise 9.4.3](https://www.splunk.com/en_us/download/splunk-enterprise.html)

## Installation de Splunk Enterprise (Serveur)

#### Mise à jour et téléchargement

```sh
apt update
wget -O splunk-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/splunk/releases/9.4.3/linux/splunk-9.4.3-237ebbd22314-linux-amd64.deb"
chmod +x splunk-9.4.3-237ebbd22314-linux-amd64.deb
dpkg -i splunk-9.4.3-237ebbd22314-linux-amd64.deb
```

#### Activer démarrage automatique + Pare-feu

```sh
/opt/splunk/bin/splunk enable boot-start
ufw enable
ufw allow OpenSSH
ufw allow 8000     # UI Web
ufw allow 9997     # Réception de logs du Forwarder
ufw status
```

#### Démarrer Splunk

```sh
/opt/splunk/bin/splunk start --accept-license
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_02.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_03.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_04.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_05.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_06.png)

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_07.png)

## Installation du Splunk Universal Forwarder (Client)

#### Télécharger et installer

- [Splunk Universal Forwarder 9.4.3](https://www.splunk.com/en_us/download/universal-forwarder.html)

```sh
wget -O splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.3/linux/splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb"
chmod +x splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
dpkg -i splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
```

#### Activer le démarrage et démarrer le service

```sh
/opt/splunkforwarder/bin/splunk enable boot-start
/opt/splunkforwarder/bin/splunk start
```

#### Ajouter le serveur Splunk Enterprise comme destination

- Remplace `<IP_Splunk_Entreprise>` par l'IP réelle de ton serveur Splunk

```sh
/opt/splunkforwarder/bin/splunk add forward-server <IP_Splunk_Entreprise>:9997 -auth admin:Admin@123
/opt/splunkforwarder/bin/splunk list forward-server
```

#### Pare-feu

```sh
ufw enable
ufw allow OpenSSH
ufw allow 9997
ufw status
```

#### Permissions recommandées

```sh
chown -R splunkfwd:splunkfwd /opt/splunkforwarder
chmod -R 755 /opt/splunkforwarder
```

#### Surveillance des logs /var/log/syslog

```sh
tail -f /var/log/syslog
```

#### Ajouter la surveillance du fichier syslog

```sh
/opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
```

## Configuration avancée : inputs.conf

#### Créer/éditer le fichier inputs.conf

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

#### Contenu du fichier inputs.conf

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

#### Redémarrer le Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

#### `Requêtes dans Search & Reporting`

```sh
index="linux_os_logs" sourcetype=syslog
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_08.png)
