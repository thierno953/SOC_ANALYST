# Investigation des attaques de force brute RDP sur les connexions Windows

#### Installation et configuration de Sysmon

> Téléchargement des composants :

> [Sysmon pour Windows](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

> [Configuration Sysmon recommandée](https://github.com/SwiftOnSecurity/sysmon-config)

#### Commandes d'installation

```sh
# Installer Sysmon avec configuration recommandée
C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

# Vérifier que le service Sysmon fonctionne
C:\Users\Administrator\Downloads\Sysmon> Get-Service Sysmon*

# Mettre à jour la configuration Sysmon
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml

# Désinstaller ou mettre à jour la configuration
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

- Visualisation des logs
  - Ouvrir l'Observateur d'événements Windows :

`Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

![Enterprise](/Splunk_Windows/assets/splunk_windows_02.png)

#### Configuration de Splunk Universal Forwarder

- Modifier `inputs.conf` pour récupérer les logs Sysmon et autres

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```sh
[WinEventLog://Application]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Application

[WinEventLog://Security]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Security

[WinEventLog://System]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:System

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog:Sysmon
renderXml = false
```

- Gestion du service Splunk Universal Forwarder

```sh
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe restart
.\splunk.exe status
.\splunk.exe list forward-server
```

#### Simulation d'attaque de force brute RDP

- Sur une machine Linux (attacker) :

```sh
# nano password.txt

password123
admin
letmein
Password1
```

```sh
apt install hydra -y
hydra -l administrator -P password.txt <IP_VICTIM_MACHINE> rdp
```

`Requêtes dans Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" SourceIp="<IP_Attach_Machine>"
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_03.png)

#### Réponse à incident

- **Blocage via interface graphique Windows Defender Firewall**

- Ouvrir : `Start > Windows Defender Firewall > Inbound Rules`

- Créer une nouvelle règle :

  - Type : Ports TCP

  - Port local spécifique : 3389

  - Action : Bloquer la connexion

  - Profils : Domain, Private, Public

  - Nommer la règle : "Block RDP Brute Force"

- **Blocage via PowerShell**

```sh
New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP_ATTACK>
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_04.png)
