# Configuration d'une machine Ubuntu avec Splunk Entreprise et Splunk Universal Forwarder

- Installer Splunk Entreprise et Universal Forwarder sur Linux.
- Configurer Universal Forwarder.
- Activer l'envoi et la réception sur le tableau de bord Splunk.

#### 1 - Installation de Splunk Entreprise

- Vérifier les informations système

```sh
root@entreprise:# uname -a
```

- Mettre à jour les paquets

```sh
root@entreprise:# sudo apt update
```

- Télécharger Splunk Entreprise

```sh
root@entreprise:# wget -O splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.3.1/linux/splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
```

- Installer Splunk Entreprise

```sh
root@entreprise:# sudo dpkg -i splunk-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
```

- Activer le démarrage automatique de Splunk

```sh
root@entreprise:# sudo /opt/splunk/bin/splunk enable boot-start
```

- Configurer le pare-feu

```sh
root@entreprise:# sudo ufw enable
root@entreprise:# sudo ufw allow OpenSSH
root@entreprise:# sudo ufw allow 8000
root@entreprise:# sudo ufw allow 9997
root@entreprise:# sudo ufw status
```

- Démarrer Splunk

```sh
root@entreprise:# sudo /opt/splunk/bin/splunk start
```

#### 2 - Installation de Splunk Universal Forwarder (UF)

- Mettre à jour les paquets

```sh
root@forwarder:# sudo apt update
```

- Télécharger Splunk Universal Forwarder

```sh
root@forwarder:# wget -O splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb "https://download.splunk.com/products/universalforwarder/releases/9.3.1/linux/splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb"
```

- Rendre le fichier exécutable

```sh
root@forwarder:# sudo chmod +x splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
```

- Installer Splunk Universal Forwarder

```sh
root@forwarder:# sudo dpkg -i splunkforwarder-9.3.1-0b8d769cb912-linux-2.6-amd64.deb
```

- Activer le démarrage automatique

```sh
root@forwarder:# sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

- Démarrer Splunk Universal Forwarder

```sh
root@forwarder:# sudo /opt/splunkforwarder/bin/splunk start
```

- Configurer le serveur de forwarding vers Splunk Entreprise

```sh
root@forwarder:# sudo /opt/splunkforwarder/bin/splunk add forward-server <IP Splunk Entreprise>:9997 -auth admin:Admin@123
```

- Vérifier les serveurs de forwarding

```sh
root@forwarder:# sudo /opt/splunkforwarder/bin/splunk list forward-server
```

- Configurer le pare-feu

```sh
root@forwarder:# sudo ufw enable
root@forwarder:# sudo ufw allow OpenSSH
root@forwarder:# sudo ufw allow 8000
root@forwarder:# sudo ufw allow 9997
```

- Configuration du fichier server.conf pour le forwarder

```sh
root@forwarder:# nano /opt/splunkforwarder/etc/system/local/server.conf

[diskUsage]
minFreeSpace = 1000
```

- Appliquer les permissions et redémarrer le service

```sh
root@forwarder:# chown -R splunkfwd:splunkfwd /opt/splunkforwarder
root@forwarder:# chmod -R 755 /opt/splunkforwarder
root@forwarder:# /opt/splunkforwarder/bin/splunk restart
```

![splunk](//Splunk-Ubuntu-Server-22/assets/02.png)

- Création d'un index pour les logs Syslog

  - a - Index
  - b - Type de source

- Accéder au répertoire des logs

```sh
root@forwarder:~$ cd /var/log/
```

- Afficher les logs en temps réel

```sh
root@forwarder:/var/log# tail -f syslog
```

- Ajouter le monitoring du fichier syslog

```sh
root@forwarder:/var/log# /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog
```

- Accéder au répertoire de configuration local

```sh
root@forwarder:/var/log# cd /opt/splunkforwarder/etc/system/local/
```

- Configuration du fichier inputs.conf pour le monitoring des logs

```sh
root@forwarder:/opt/splunkforwarder/etc/system/local# nano inputs.conf

[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog
```

- Redémarrer le service Splunk

```sh
root@forwarder:/opt/splunkforwarder/etc/system/local# cd ../../../..
root@forwarder:/opt# /opt/splunkforwarder/bin/splunk restart
```

![splunk](/Splunk-Ubuntu-Server-22/assets/03.png)

##### Recherche et reporting

- Settings --> Indexes --> New Index
- Settings --> Source types --> New Source Type

```sh
index="linux_os_logs"
index="linux_os_logs" sourcetype=syslog
index="linux_os_logs" sourcetype=syslog
```
