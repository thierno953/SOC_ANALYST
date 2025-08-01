# Surveillance de l'intégrité des fichiers sensibles

- Installer et configurer auditd pour la surveillance des fichiers sensibles

- Configurer l'envoi des logs vers Splunk

- Simuler une attaque (modification non autorisée)

- Analyser les logs et répondre à l'incident

#### Installation et configuration de auditd

```sh
apt-get update
apt-get install auditd -y
systemctl start auditd
systemctl status auditd
```

#### Configuration des règles d'intégrité :

```sh
nano /etc/audit/rules.d/audit.rules
```

#### Contenu de audit.rules

```sh
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1

## Surveiller les modifications dans /etc/
-w /etc/ -p wa -k file_integrity
```

#### Appliquer les règles

```sh
systemctl restart auditd
auditctl -l   # Vérifier que la règle est bien en place
```

## Configuration de Splunk Universal Forwarder

#### Modifier inputs.conf

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

#### Contenu du fichier inputs.conf

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
index = linux_file_integrity
sourcetype = auditd

[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

#### Redémarrer Splunk Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulation d'une attaque (modification non autorisée)

#### Créer ou modifier un fichier dans /etc/

```sh
nano /etc/myfirstfile.txt

# Contenu du fichier
This is my first file.
```

#### Surveiller les logs en temps réel

```sh
tail -f /var/log/audit/audit.log
```

#### Rechercher les logs liés à la clé file_integrity

```sh
ausearch -k file_integrity
ausearch -k file_integrity | grep myfirstfile
```

> `Requêtes dans Search & Reporting`

- Utilise ces requêtes dans l’interface Splunk pour analyser

```sh
index=linux_file_integrity sourcetype=auditd "myfirstfile"
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_12.png)

```sh
index=linux_file_integrity sourcetype=auditd key=file_integrity
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_13.png)
