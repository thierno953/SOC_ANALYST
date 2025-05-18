# Investigation des modifications suspectes du Registre Windows

#### Vérification préalable

- **Vérifier que Sysmon est installé** avec une configuration qui collecte les événements liés au Registre (EventID 12, 13, 14).

- **Confirmer que Splunk Universal Forwarder collecte bien les logs Sysmon** (`index=sysmon_logs`).

- Ouvrir l’éditeur de Registre : `Start > Registry Editor`

#### Simulation d’attaque : modification malveillante du Registre

> Simulation de modification malveillante du Registre

```sh
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"

# Création d'une clé de démarrage automatique
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe" -PropertyType String

# Vérification de la création
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

- `Recherche générale des événements Registre Sysmon`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" EventID=12 OR EventID=13 OR EventID=14
```

- `Requêtes dans Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*malwaretest*"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "CurrentVersion\Run"
```

#### Réponse à incident : suppression de la clé malveillante

```sh
# Suppression de la clé malveillante
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"

# Vérification de la suppression
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

#### Mesures de protection à long terme

- Configuration de règles AppLocker

```sh
# Exemple de politique pour restreindre les exécutables
New-AppLockerPolicy -RuleType Publisher,Hash,Path -FilePath "C:\Policy.xml" -User Everyone
Set-AppLockerPolicy -XmlPolicy "C:\Policy.xml"
```

#### Surveillance renforcée avec Sysmon

- Ajouter dans la configuration Sysmon la surveillance des clés critiques du Registre

```sh
<RuleGroup name="" groupRelation="or">
    <RegistryEvent onmatch="include">
        <TargetObject condition="contains">CurrentVersion\Run</TargetObject>
        <TargetObject condition="contains">Winlogon\Shell</TargetObject>
        <TargetObject condition="contains">Policies\Explorer\Run</TargetObject>
    </RegistryEvent>
</RuleGroup>
```

#### Alerte Splunk pour détection automatique

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" (EventID=12 OR EventID=13 OR EventID=14)
("CurrentVersion\Run" OR "Winlogon\Shell" OR "Policies\Explorer\Run")
```
