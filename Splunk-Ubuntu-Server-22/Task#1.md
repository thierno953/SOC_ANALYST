# Détection d'accès non autorisés sur Linux bloqués par Fail2Ban

- Installer Fail2Ban
- Configurer Fail2Ban
- Simuler une attaque
- Analyser les logs

#### Installation et configuration de Fail2Ban

- Mettre à jour les paquets

```sh
root@forwarder:~$ sudo apt update
```

- Installer Fail2Ban

```sh
root@forwarder:~$ sudo apt install fail2ban -y
```

- Configuration de Fail2Ban pour SSH

```sh
root@forwarder:~$ sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 600
findtime = 600
```

- Ajouter le monitoring des logs Fail2Ban dans Splunk et redémarrer les services

```sh
root@forwarder:~$ /opt/splunkforwarder/bin/splunk add monitor /var/log/fail2ban.log
root@forwarder:~$ sudo systemctl restart fail2ban
root@forwarder:~$ sudo fail2ban-client status
root@forwarder:~$ nano /var/log/fail2ban.log
```

- Configuration des inputs.conf pour Splunk

```sh
root@forwarder:~$ sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

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
sourcetype = fail2ban
index = fail2ban_logs
```

- Redémarrer Splunk pour appliquer les changements

```sh
root@forwarder:~$ sudo /opt/splunkforwarder/bin/splunk restart
```

#### Simulation d'une attaque SSH par force brute

- Mettre à jour les paquets

```sh
root@attack:~$ sudo apt update
```

- Installer Hydra (outil de force brute)

```sh
root@attack:~$ sudo apt install hydra -y
```

- Créer un fichier de mots de passe pour l'attaque

```sh
root@attack:~$ nano passwords.txt
# Contenu du fichier :
admin
test
test2
test1000
password
passme
meme
meraku
herko
```

- Lancer l'attaque SSH par force brute

```sh
root@attack:~$ hydra -l admin -P passwords.txt <SPLUNK-IP> ssh
```

- Surveiller les logs Fail2Ban en temps réel

```sh
root@forwarder:~$ nano /var/log/fail2ban.log
root@forwarder:~$ tail -f /var/log/fail2ban.log
```

##### Analyse des logs dans Splunk

- Recherche et reporting

```sh
index="fail2ban_logs"
index="fail2ban_logs" src="<IP ATTACK>"
index="fail2ban_logs" sourcetype="fail2ban" src="<IP ATTACK>"
index="fail2ban_logs" | search "<IP ATTACK>"
```
