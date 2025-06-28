# Enquête sur les attaques par force brute RDP sur les connexions Windows

#### Objectif

- Détecter une attaque par force brute via RDP sur un hôte Windows.

- Utiliser Sysmon pour journaliser les connexions.

- Utiliser ELK pour visualiser les événements.

#### Installation de Sysmon

- Télécharger Sysmon et le fichier de configuration :

  - [Sysmon (Microsoft)](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

  - [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)

#### Emplacement recommandé : `C:\Sysmon\`

- Commandes PowerShell

```sh
# Se placer dans le dossier Sysmon
cd C:\Sysmon\

# Installer Sysmon avec la configuration
.\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

# Vérifier le service
Get-Service Sysmon*

# Mettre à jour la configuration
.\Sysmon64.exe -c .\sysmonconfig-export.xml

# Relancer avec une configuration mise à jour
.\Sysmon64.exe -u .\sysmonconfig-export.xml
```

#### Fichier de log

- Chemin : `Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

#### Intégrer à ELK (Fleet/Elastic Agent)

- Dans l’interface Kibana :

- Aller dans :

`Management > Integrations > Windows`

- Vérifier que les sources suivantes sont **actives** :

  - Forwarded

  - PowerShell

  - PowerShell Operational

  - Sysmon Operational

- Ces journaux doivent être collectés par l’Elastic Agent installé sur la machine cible

#### Simulation d’attaque RDP

- Machine attaquante (Ubuntu)

```sh
# Installer hydra
apt install hydra -y

# Lancer une attaque brute-force RDP
hydra -l administrator -P password.txt <IP_FLEET_AGENT> rdp
```

#### Visualisation dans Kibana (ELK)

- Interface : `Analytics > Discover`
  > Filtres à utiliser

```sh
winlog.channel:"Microsoft-Windows-Security-Auditing"
event.code: 3 and source.ip: "<IP_ATTACKER>"
event.code: 3 and source.ip:"<IP_ATTACKER>" and rule.name: "RDP"
```
