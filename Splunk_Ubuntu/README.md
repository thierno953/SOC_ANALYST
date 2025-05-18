# Investigation de sécurité d'une machine Ubuntu avec Splunk

- [Splunk Enterprise Previous Releases](https://www.splunk.com/en_us/download/previous-releases.html?locale=en_us)

![Enterprise](/assets/splunk_linux_01.png)

## Installation de Splunk Enterprise (Serveur)

#### Mise à jour et téléchargement

```sh
apt update
wget -O splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.3.1/linux/splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
chmod +x splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
dpkg -i splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
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
/opt/splunk/bin/splunk start
```

![Enterprise](/assets/splunk_linux_02.png)
![Enterprise](/assets/splunk_linux_03.png)
![Enterprise](/assets/splunk_linux_04.png)
![Enterprise](/assets/splunk_linux_05.png)
![Enterprise](/assets/splunk_linux_06.png)
![Enterprise](/assets/splunk_linux_07.png)

## Installation du Splunk Universal Forwarder (Client)

#### Télécharger et installer

- [Splunk Universal Forwarder Previous Releases](https://www.splunk.com/en_us/download/previous-releases-universal-forwarder.html)

```sh
wget -O splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.3.1/linux/splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
chmod +x splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
dpkg -i splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
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

#### Accès au répertoire

```sh
cd /opt/splunkforwarder/etc/system/local/
```

#### Créer/éditer le fichier inputs.conf

```sh
nano inputs.conf
```

#### Contenu du fichier inputs.conf

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog


# Ont peux ajouter d'autres logs ici, comme `auth.log` ou `kern.log`

[monitor:///var/log/auth.log]
disabled = false
index = linux_os_logs
sourcetype = authlog

[monitor:///var/log/kern.log]
disabled = false
index = linux_os_logs
sourcetype = kernlog
```

#### Redémarrer le Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Configuration sur l’interface Web Splunk

#### Créer un index personnalisé

- Aller dans `Settings > Indexes > New Index`

- Nom : `linux_os_logs`

#### (Optionnel) Créer un nouveau sourcetype

- `Settings > Source types > New Source Type`

- **Exemple** : `syslog`, `authlog`, `kernlog`

> `Requêtes dans Search & Reporting`

```sh
index="linux_os_logs"
index="linux_os_logs" sourcetype=syslog
index="linux_os_logs" sourcetype=authlog "Failed password"
index="linux_os_logs" | stats count by host, sourcetype
```
