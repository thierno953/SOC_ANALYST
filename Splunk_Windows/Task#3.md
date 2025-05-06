# Investigation des modifications suspectes du Registre Windows

> Vérifier que Sysmon est installé avec une configuration incluant le monitoring du Registre

> S'assurer que la configuration Splunk inclut bien les logs Sysmon

> `Start > Registry Editor`

#### Simulation d'attaque et analyse

> Simulation de modification malveillante du Registre

```sh
PS C:\Users\Administrator> New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"

# Création d'une clé de démarrage automatique
PS C:\Users\Administrator> New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe" -PropertyType String

# Vérification de la création
PS C:\Users\Administrator> Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

> `Recherche générale des modifications du Registre`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" EventID=12 OR EventID=13 OR EventID=14
```

> `Requêtes dans Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*malwaretest*"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "CurrentVersion\Run"
```

#### Réponse à incident

```sh
# Suppression de la clé malveillante
PS C:\Users\Administrator> Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"

# Vérification de la suppression
PS C:\Users\Administrator> Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

#### Mesures de protection à long terme

> Configuration de règles AppLocker

```sh
# Exemple de politique pour restreindre les exécutables
New-AppLockerPolicy -RuleType Publisher,Hash,Path -FilePath "C:\Policy.xml" -User Everyone
Set-AppLockerPolicy -XmlPolicy "C:\Policy.xml"
```

- Surveillance avec règles Sysmon supplémentaires
  > Ajouter dans la configuration Sysmon :

```sh
<RuleGroup name="" groupRelation="or">
    <RegistryEvent onmatch="include">
        <TargetObject condition="contains">CurrentVersion\Run</TargetObject>
        <TargetObject condition="contains">Winlogon\Shell</TargetObject>
        <TargetObject condition="contains">Policies\Explorer\Run</TargetObject>
    </RegistryEvent>
</RuleGroup>
```

> Alerte Splunk pour détection automatique pour les modifications critiques :

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" (EventID=12 OR EventID=13 OR EventID=14)
("CurrentVersion\Run" OR "Winlogon\Shell" OR "Policies\Explorer\Run")
```
