# Detecting Suspicious Activities Using SysmonForLinux

#### Installation de SysmonForLinux

- Lien officiel [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
# Enregistrer la clé et le dépôt Microsoft
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# Mettre à jour et installer Sysmon
sudo apt-get update
sudo apt-get install sysmonforlinux
```

> Créer un fichier de configuration personnalisé

```sh
nano sysmon-config.xml
```

- Exemple de configuration [MSTIC Sysmon Resources](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

```sh
# Installer la configuration
sysmon -i sysmon-config.xml

# Redémarrer le service
systemctl restart sysmon
systemctl status sysmon

# Vérifier les logs
tail -f /var/log/syslog | grep sysmon
```

#### Préparation d’ELK pour la détection

- **Kibana** → `Management > Integrations`

- Rechercher : `Sysmon for Linux`

- Ajouter l’intégration et configurer :

  > Paths : `/var/log/syslog*`

  > Associer à une **Agent Policy**

- Valider → Dashboards disponibles : `[Sysmon] Sysmon for Linux Logs Overview`

> Kibana Discover (recherche rapide) :

```sh
process.name: sysmon
```

#### Simulation d’une activité malveillante

> Simuler une configuration malicieuse [MalwareBazaar](https://bazaar.abuse.ch/)

```sh
touch /etc/apt/apt.conf.d/99-suspicious-config
sudo bash -c "echo 'malicious config' > /etc/apt/apt.conf.d/99-malicious-config"

# Télécharger une URL connue pour héberger du malware (simulé, pas d’exécution réelle ici)
curl -X GET "https://bazaar.abuse.ch/" -v
```

- `Visualiser les événements dans Kibana`

```sh
process.name: sysmon and message: "99"
process.name: sysmon and message: "bazaar.abuse.ch"
```

#### Réponse à l’incident

> Identifier et supprimer les fichiers suspects

```sh
find / -name "99-malicious-config" -type f
rm -rf /etc/apt/apt.conf.d/99-suspicious-config
rm -rf /etc/apt/apt.conf.d/99-malicious-config
find / -name "99-malicious-config" -type f
```

> Vérifier les connexions réseau actives

```sh
netstat -ltnp
```
