# Investigation des abus PowerShell sur les machines Windows

#### Vérification de l'installation Sysmon

> `Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

> Fichier `: C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

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

> Redémarrage du service Splunk

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

> `Requêtes dans Search & Reporting`

#### Simulation d'attaque et analyse

> [Téléchargement du fichier test EICAR](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

#### Requêtes d'investigation dans Splunk

> `Recherche d'activité PowerShell générale`

```sh
index="powershell_logs" sourcetype="WinEventLog:PowerShell"
```

> `Recherche spécifique de l'attaque`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*eicar*"
index="powershell_logs" sourcetype="WinEventLog:PowerShell" "*eicar*"
```

#### Réponse à incident

`Start > Windows Defender Firewall > Outbound Rules > New Rules > Program > All programs > Block the connection > (Domain, Private, and Public) > Provide a Name for the "Block All Traffi"`

> PowerShell

```sh
# Blocage complet du trafic sortant
New-NetFirewallRule -DisplayName "Block All Traffic" -Direction Outbound -Action Block

# Alternative : Blocage spécifique à PowerShell
New-NetFirewallRule -DisplayName "Block PowerShell Internet Access" -Program "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -Direction Outbound -Action Block
```
