# Investigation des modifications suspectes du Registre Windows

#### Vérification préalable

- **Vérifier que Sysmon est installé** avec une configuration qui collecte les événements liés au Registre (EventID 12, 13, 14).

- **Confirmer que Splunk Universal Forwarder collecte bien les logs Sysmon** (`index=sysmon_logs`).

- Ouvrir l’éditeur de Registre : `Start > Registry Editor`

#### Simulation d’attaque : modification malveillante du Registre

> Simulation de modification malveillante du Registre

```sh
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"

# Vérification de la création
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" | Select-Object MalwareTest
echo 'echo malwaretest' > C:\malwaretest.exe
```

- `Requêtes dans Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*malwaretest*"
```

#### Réponse à incident : suppression de la clé malveillante

```sh
# Suppression de la clé malveillante
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"

# Vérification de la suppression
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```
