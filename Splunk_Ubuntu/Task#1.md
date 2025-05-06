# Détection d'accès non autorisé par Fail2Ban

- Détection d'accès non autorisé via SSH sur Linux à l'aide de Fail2Ban, avec envoi des logs vers Splunk via Universal Forwarder.

## Installer et configurer Fail2Ban sur la machine cible

#### Mise à jour & installation de Fail2Ban

```sh
apt update
apt install fail2ban -y
```

#### Configuration de la prison SSH (jail.local)

```sh
nano /etc/fail2ban/jail.local
```

#### Contenu du fichier :

```sh
[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

#### Redémarrer et vérifier l’état de Fail2Ban

```sh
systemctl restart fail2ban
fail2ban-client status
tail -f /var/log/fail2ban.log
```

## Configurer le Splunk Universal Forwarder pour collecter les logs

#### Ajout du suivi des fichiers de log

```sh
/opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
```

#### Modifier inputs.conf pour ajouter les logs Fail2Ban et SSH

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

#### Contenu du fichier :

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = false
index = security_incidents
sourcetype = linux_secure
whitelist = Failed|invalid|Denied

[monitor:///var/log/fail2ban.log]
disabled = false
index = fail2ban_logs
sourcetype = fail2ban
```

#### Redémarrage du Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simuler une attaque SSH brute force

#### Sur la machine attacker, installer Hydra

```sh
apt update
apt install hydra -y
```

#### Lancer une attaque brute force SSH

```sh
hydra -l admin -P passwords.txt <IP_VICTIME> ssh
```

- Assurez-vous que `admin` est un utilisateur existant et que le port SSH est accessible.

#### Observer les logs sur la machine victime

```sh
tail -f /var/log/fail2ban.log
```

## Analyse des événements sur Splunk

#### Accéder à Splunk Web : `http://<IP_SPLUNK_SERVER>:8000`

> `Requêtes dans Search & Reporting`

```sh
index="fail2ban_logs"
```

```sh
index="fail2ban_logs" sourcetype="fail2ban" src="<IP_ATTACKER>"
```

```sh
index="fail2ban_logs" | search "<IP_ATTACKER>"
```

```sh
index="security_incidents" sourcetype="linux_secure" "Failed password"
```

```sh
index="fail2ban_logs" | stats count by src, action
```
