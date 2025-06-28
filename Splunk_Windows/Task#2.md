# Investigation des abus PowerShell sur les machines Windows

#### Vérification de la collecte des logs PowerShell et Sysmon

- Visualisation des logs PowerShell
  - Ouvrir l’Observateur d’événements Windows

`Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

#### Configuration Splunk Universal Forwarder

- Fichier `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```sh
[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = powershell_logs
sourcetype = WinEventLog:PowerShell
renderXml = false

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

- Redémarrage du service Splunk

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

#### Simulation d’attaque PowerShell

> [Télécharger un fichier de test EICAR (test anti-malware standard) pour simuler une activité suspecte](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

#### Requêtes d’investigation dans Splunk

`Recherche d'activité PowerShell générale`

```sh
index="powershell_logs" sourcetype="WinEventLog:PowerShell"
```

`Recherche spécifique liée au fichier EICAR`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*eicar*"
```

#### Réponse à incident

- **Blocage via Windows Defender Firewall (sortant)**

- Interface graphique :

  - `Start > Windows Defender Firewall > Outbound Rules > New Rule`

  - Type : Program

  - Cible : Tous les programmes ou uniquement PowerShell

  - Action : Bloquer la connexion

  - Profils : Domain, Private, Public

  - Nommer la règle, par exemple : "Block PowerShell Internet Access"

- **Blocage via PowerShell**

```sh
# Bloquer tout le trafic sortant (à utiliser avec précaution)
New-NetFirewallRule -DisplayName "Block All Traffic" -Direction Outbound -Action Block
```
