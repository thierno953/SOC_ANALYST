# Surveillance des modifications du registre Windows

#### Objectif

- Détecter la création ou la modification de clés dans la base de registre Windows.

- Utiliser Sysmon et ELK pour surveiller les manipulations suspectes.

- Réagir à une tentative de persistance typique via la clé

  `HKCU:\Software\Microsoft\Windows\CurrentVersion\Run`.

#### Vérifier l’intégration dans ELK

- **Dans Kibana**

- `Management > Fleet > Agents` : vérifier que l'agent Fleet est en ligne sur la machine cible (Windows).

- `Management > Integrations > integration policies` :

  - **Windows**

  - **Sysmon Operational** doit être **actif**.

  - S’assurer que l’événement **Sysmon Event ID 13** est collecté.

#### Simulation d'une attaque : persistance via la base de registre

- Commande PowerShell

```sh
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest1" -Value "C:\malwaretest1.exe"
```

- Cette commande ajoute une entrée de persistance dans la clé `Run`, typique des malwares

#### Visualisation dans Kibana 

- Interface : `Analystics > Discover`

```sh 
event.code:13
```

![ELK](/Elastic_Stack_Windows/assets/04.png)

- **Sysmon Event ID 13** : Activité sur une clé de registre (création/modification).

#### Incident Response

- Étapes de réponse :

  - **Vérifier** l’origine de l’ajout dans la base de registre.
 
  - **Supprimer** la clé suspecte si elle est malveillante

```sh
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest1"
```
