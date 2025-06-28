# Forensics Windows

- La pratique consistant à collecter et analyser des données provenant de systèmes Windows pour identifier, enquêter et se remettre d’incidents de sécurité.

- **Objectifs** :

  - Identifier les activités malveillantes
  - Récupérer des preuves
  - Prévenir d'autres attaques

- **Domaines d’analyse principaux** : `système de fichiers, registre, journaux d’événements, mémoire, activité réseau`.

# Vue d'ensemble du processus d'intervention sur incident

#### Composants clés de Forensics Windows

- **Système de fichiers** : analyser les fichiers, dossiers, horodatages et données cachées.

- **Registre** : suivre les modifications de configuration, l'activité utilisateur et les paramètres d’applications.

- **Journaux d’événements** : examiner les journaux Sécurité, Application et Système pour détecter les événements pertinents.

- **Mémoire (RAM)** : capturer les données volatiles, y compris les processus en cours et les connexions réseau.

#### Pourquoi Forensics Windows est importante

- **Réponse aux incidents** : détecter et analyser l’étendue et l’impact d’une compromission.

- **Preuve légale** : garantir l'intégrité des données pour qu’elles soient recevables au tribunal.

- **Récupération** : aider à restaurer les systèmes à un état opérationnel après une attaque.

- **Sécurité proactive** : améliorer les défenses en apprenant des incidents passés.

#### Pourquoi utiliser PowerShell pour Forensics

- **Outil en ligne de commande puissant** : intégré à Windows, donc facilement accessible.

- **Automatisation** : permet d’automatiser les tâches récurrentes d’analyse et de réponse aux incidents.

- **Accès au système** : accès direct aux fichiers, processus, journaux et réseau, sans outils tiers.

- **Souplesse de script** : possibilité de créer des scripts personnalisés pour les besoins spécifiques.

#### Cmdlets PowerShell clés pour Forensics

- `Get-Process` : lister tous les processus en cours avec détails (mémoire, CPU, etc.).
- `Get-EventLog` : récupérer et filtrer les journaux d’événements Windows.
- `Get-FileHash` : générer des hachages (MD5, SHA256) pour vérifier l’intégrité des fichiers.
- `Get-ChildItem` : lister récursivement les fichiers/dossiers avec métadonnées.
- `Get-WmiObject` : accéder aux infos système (logiciels installés, éléments de démarrage, configurations...).

#### 1 - Collecte d'informations système avec PowerShell

```sh
Get-ComputerInfo
Get-ComputerInfo | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\SystemInfo.txt
Get-Process
Get-Process | Format-Table -AutoSize
Get-Process | Format-Table -AutoSize | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessList.txt
```

#### 2 - Enquête sur les comptes utilisateurs

- Lister tous les comptes AD

```sh
Get-ADUser -Filter *
Get-ADUser -Filter * | Select-Object Name, Enabled | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ADUserAccounts.txt
```

- Lister les membres du groupe "Domain Admins" :

```sh
Get-ADGroupMember -Identity "Domain Admins"
```

- Obtenir les détails d’un utilisateur AD spécifique :

```sh
Get-ADUser -Identity "thierno" -Properties *
```

#### 3 - Analyse des processus

```sh
Get-Process

Get-Process | Where-Object { $_.Name -notin @('explorer', 'svchost', 'winlogon', 'lsass', 'services')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousProcesses.txt"

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindows.txt

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindowsBrief.txt

Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU

Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\HighCPUProcesses.txt

Get-Process | Sort-Object PM -Descending | Select-Object -First 10 Name, Id, PM
```

#### 4 - Analyse des services

```sh
Get-Service

Get-Service | Format-Table -AutoSize

Get-Service | Where-Object { $_.Name -notin @('TrustedInstaller', 'WindDefend', 'EventLog', 'Dhcp', 'Dnscache')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousServices.txt"

Get-Service -Name "BITS" | Format-List *

Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName

Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ServiceOutOfWindows.txt
```

#### 5 - Analyse des tâches planifiées

```sh
Get-ScheduledTask | Format-Table -AutoSize

Get-ScheduledTask | Where-Object { $_.TaskName -notin @('UpdateTask', 'SystemTasks', 'WindowsTasks')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousScheduledTasks.txt"
```

#### 6 - Analyse des connexions Internet actives

```sh
Get-NetTCPConnection

Get-NetTCPConnection | Format-Table -AutoSize

Get-NetTCPConnection | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ActiveNetTCPConnections.txt"

Get-NetTCPConnection -LocalPort 3389

Get-NetTCPConnection -LocalPort 3389 | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\RDPConnection.txt"

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses.txt"

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses1.txt"
```
