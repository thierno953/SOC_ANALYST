# Surveillance de l'activité des comptes utilisateurs

- Installer Sysmon for Linux

- Configurer l’ingestion des logs avec Splunk

- Simuler une activité malveillante

- Visualiser les événements dans Splunk

- Réagir à l’incident

#### Installateur Sysmon pour Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# Ajouter le dépôt Microsoft
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install sysmonforlinux -y
```

#### Créer et appliquer un fichier de configuration Sysmon

```sh
# Télécharger ou copier un fichier de configuration XML personnalisé
nano sysmon-config.xml

# Exemple de ligne de base (à adapter selon les besoins)
<RuleGroup name="default" groupRelation="or">
  <ProcessCreate onmatch="include" />
  <FileCreateTime onmatch="include" />
  <NetworkConnect onmatch="include" />
</RuleGroup>
```

- [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
# Appliquer la configuration
sysmon -i sysmon-config.xml

# Vérification du statut
systemctl restart sysmon
systemctl status sysmon
```

#### Configurer Splunk Universal Forwarder

```sh
# Vérifier la présence de logs sysmon dans syslog
grep -i sysmon /var/log/syslog
```

```sh
# Modifier la configuration d’entrée de Splunk
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/auth.log]
disabled = false
index = linux_os_logs
sourcetype = linux_secure
```

- Modifier `outputs.conf` si nécessaire

```sh
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <IP_INDEXEUR>:9997
```

- **NB** : Ajout recommandé dans Splunk Web `Apps > Browse More Apps > Splunk Add-on for Sysmon for Linux`

```sh
# Redémarrer le Forwarder Splunk
/opt/splunkforwarder/bin/splunk restart
```

#### Simuler une activité malveillante

```sh
# Création d’un faux utilisateur
adduser maluser --disabled-password --gecos ""
echo "maluser:Password123!" | chpasswd
```

```sh
# Vérifier les traces dans syslog
tail -f /var/log/syslog | grep maluser
tail -f /var/log/auth.log | grep maluser
```

> `Requêtes dans Search & Reporting`

```sh
index="linux_os_logs"
index="linux_os_logs" sourcetype=syslog "Process Create"
index="linux_os_logs" sourcetype=syslog CommandLine="*adduser*"
index="linux_os_logs" sourcetype=syslog User="maluser"
index="linux_os_logs" process=sysmon maluser
index="linux_os_logs" sourcetype=linux_secure "Accepted password for"
```

#### Création d'une alerte dans Splunk

```sh
index="linux_os_logs" sourcetype=syslog CommandLine="/usr/sbin/adduser*"
```

#### Réponse à l’incident

- Vérifier et supprimer le compte malveillant

```sh
less /etc/passwd
sudo chage -l maluser
sudo deluser --remove-home maluser
```
