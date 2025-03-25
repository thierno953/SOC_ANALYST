# Surveillance de l'intégrité des fichiers pour les répertoires sensibles

- **Objectif** : Surveiller les modifications non autorisées dans les répertoires sensibles à l'aide d'Auditd et analyser les logs dans Splunk.
- **Étapes** :
  - Installer et configurer Auditd.
  - Configurer Splunk pour surveiller les logs d'Auditd.
  - Simuler une tentative de modification non autorisée.
  - Analyser les logs dans Splunk.

#### 1 - Installation et configuration d'Auditd

- Mettre à jour les paquets

```sh
root@forwarder:$ apt-get update
```

- Installer Auditd

```sh
root@forwarder:$ apt-get install auditd
```

- Démarrer le service Auditd

```sh
root@forwarder:$ systemctl start auditd
```

- Vérifier le statut d'Auditd

```sh
root@forwarder:$ systemctl status auditd
```

- Accéder au répertoire des règles d'Auditd

```sh
root@forwarder:$ cd /etc/audit/rules.d
root@forwarder:$ ls
```

- Configuration des règles d'Auditd
  - Éditer le fichier de règles d'Auditd

```sh
root@forwarder:/etc/audit/rules.d# nano audit.rules
```

- Coller le contenu suivant dans audit.rules

```sh
## Première règle - supprimer toutes les règles existantes
-D

## Augmenter la taille des buffers pour gérer les événements stressants
## Augmenter cette valeur pour les systèmes très actifs
-b 8192

## Définir le temps d'attente pour les événements en rafale
--backlog_wait_time 60000

## Définir le mode d'échec sur syslog
-f 1

## Surveiller les modifications dans le répertoire /etc
-w /etc/ -p wa -k file_integrity
```

- Appliquer les règles et redémarrer Auditd
  - Revenir au répertoire racine

```sh
root@forwarder:/etc/audit/rules.d# cd ../../..
```

- Redémarrer le service Auditd

```sh
root@forwarder:/# systemctl restart auditd
```

- Lister les règles actives d'Auditd

```sh
root@forwarder:/# auditctl -l
```

#### 2 - Configuration de Splunk pour surveiller les logs d'Auditd

- Éditer le fichier de configuration des inputs de Splunk

```sh
root@forwarder:/# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Ajouter ou modifier les sections suivantes

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = auditd
index = linux_file_integrity

[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

- Redémarrer Splunk pour appliquer les changements

```sh
root@forwarder:/# /opt/splunkforwarder/bin/splunk restart
root@forwarder:/# /opt/splunk/bin/splunk restart
```

![splunk](/assets/04.png)
![splunk](/assets/05.png)
![splunk](/assets/06.png)
![splunk](/assets/07.png)

#### 3 - Simulation d'une tentative de modification non autorisée et analyse des logs dans Splunk

- Vérifier les règles actives d'Auditd

```sh
root@forwarder:/# auditctl -l
```

- Créer un fichier de test dans /etc

```sh
root@forwarder:~# nano /etc/myfirstfile.txt
```

- Ajouter le contenu suivant

```sh
This is my first file.
```

- Surveiller les logs d'Auditd en temps réel

```sh
root@forwarder:/# tail -f /var/log/audit/audit.log
```

Rechercher des événements liés à l'intégrité des fichiers

```sh
root@forwarder:~# ausearch -k file_integrity
root@forwarder:~# ausearch -k file_integrity | grep myfirstfile
```

![splunk](/assets/08.png)
![splunk](/assets/09.png)
![splunk](/assets/10.png)
![splunk](/assets/11.png)

##### Recherche et reporting dans Splunk

```sh
index="linux_file_integrity" sourcetype=auditd
index="linux_file_integrity" sourcetype=auditd "myfirstfile"
index=linux_file_integrity sourcetype=auditd key=file_integrity
```
