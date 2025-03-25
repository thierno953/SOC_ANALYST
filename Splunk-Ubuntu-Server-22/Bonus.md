- Automatisation de la configuration des outils:

```sh
#!/bin/bash

# Mettre à jour les paquets
apt-get update

# Installer Sysmon
curl -s https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -o packages-microsoft-prod.deb
dpkg -i packages-microsoft-prod.deb
apt-get update
apt-get install sysmonforlinux -y

# Télécharger et appliquer la configuration Sysmon
wget -O /tmp/sysmon-config.xml https://votre-lien-vers-la-configuration-sysmon.xml
sysmon -i /tmp/sysmon-config.xml

# Redémarrer le service Sysmon
systemctl restart sysmon
systemctl enable sysmon
```

- Automatisation des tâches de surveillance

```sh
# Ouvrir le fichier crontab
crontab -e

# Ajouter une tâche pour surveiller les logs toutes les heures
0 * * * * /usr/bin/tail -n 100 /var/log/syslog | grep "maluser" >> /var/log/suspect_activity.log
```

- Création de dashboards Splunk

```sh
index="linux_os_logs" sourcetype="syslog" "Failed password" | stats count by src_ip, user | sort - count
```

- Isolation d'une machine compromise

```sh
#!/bin/bash

# Bloquer une IP suspecte
iptables -A INPUT -s <IP_SUSPECTE> -j DROP
iptables -A OUTPUT -d <IP_SUSPECTE> -j DROP
```

- Collecte de preuves

```sh
#!/bin/bash

# Créer un dossier pour stocker les preuves
mkdir -p /var/evidence/$(date +%Y-%m-%d)

# Sauvegarder les logs
cp /var/log/syslog /var/evidence/$(date +%Y-%m-%d)/syslog.log
cp /var/log/auth.log /var/evidence/$(date +%Y-%m-%d)/auth.log

# Capturer l'état des processus
ps aux > /var/evidence/$(date +%Y-%m-%d)/processes.txt

# Capturer les connexions réseau
netstat -tuln > /var/evidence/$(date +%Y-%m-%d)/network_connections.txt
```

- Notification automatique

```sh
#!/bin/bash

# Envoyer une alerte par email
echo "Alerte : Activité suspecte détectée sur $(hostname)" | mail -s "Alerte de sécurité" admin@example.com
```
