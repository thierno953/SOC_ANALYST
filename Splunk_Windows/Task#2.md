# Investigation des abus PowerShell sur les machines Windows

#### Vérification de la collecte des logs PowerShell et Sysmon

- Visualisation des logs PowerShell
  - Ouvrir l’Observateur d’événements Windows

`Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

![Enterprise](/Splunk_Windows/assets/splunk_windows_05.png)

#### Configuration Splunk Universal Forwarder

- Fichier `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

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

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = powershell_logs
sourcetype = WinEventLog:PowerShell
renderXml = false
```

- Redémarrage du service Splunk

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

#### Simulation d’attaque PowerShell

> [Télécharger un fichier de test EICAR (test anti-malware standard) pour simuler une activité suspecte](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
PS C:\Users\Administrator> schtasks /create /tn "MaliciousTasks" /tr "C:\Malware.exe" /sc once /st 12:00
PS C:\Users\Administrator> schtasks /run /tn "MaliciousTasks"
```

#### Requêtes d’investigation dans Splunk

`Recherche d'activité PowerShell générale`

```sh
index="powershell_logs" sourcetype="WinEventLog:PowerShell"
```

`Recherche spécifique liée au fichier EICAR`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*eicar*"
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_06.png)

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

![Enterprise](/Splunk_Windows/assets/splunk_windows_07.png)
