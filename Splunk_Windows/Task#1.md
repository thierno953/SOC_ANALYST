# Investigation des attaques de force brute RDP sur les connexions Windows

#### Installation et configuration de Sysmon

> Téléchargement des composants :

> [Sysmon pour Windows](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

> [Configuration Sysmon recommandée](https://github.com/SwiftOnSecurity/sysmon-config)

#### Commandes d'installation

```sh
# Installation avec configuration SwiftOnSecurity
C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

# Vérification du service
C:\Users\Administrator\Downloads\Sysmon> Get-Service Sysmon*

# Mise à jour de configuration
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

> Visualisation des logs

> `Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

#### Configuration de Splunk

> `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`

```sh
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <IP_INDEXEUR>:9997

[tcpout-server://<IP_INDEXEUR>:9997]
```

> `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```sh
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog:Sysmon
renderXml = false

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
```

> Gestion du service Splunk

```sh
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe restart
.\splunk.exe status
.\splunk.exe list forward-server
```

#### Simulation d'attaque et visualisation

> Simulation avec Hydra (machine attaquante Linux)

```sh
# Sur la machine d'attaque (Linux)
apt install hydra -y
hydra -l administrator -P password.txt <IP_VICTIME> rdp
```

> `Requêtes dans Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" sourceIp="<IP_Attach_Machine>"
```

#### Réponse à incident

> Blocage via Pare-feu Windows

`Start > Windows Defender Firewall > Inbound Rules > New Rules > Ports > TCP (Specific local ports: 3389) > Block the connection > (Domain, Private, and Public) > Provide a Name for the "Block RDP Brute Force"`

> Alternative en PowerShell

`New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP_ATTACK>`
