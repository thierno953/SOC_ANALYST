# Enquête sur les abus de PowerShell sur les machines Windows

#### Objectif

- Détecter une utilisation malveillante de PowerShell.

- Utiliser ELK avec Sysmon et Windows Event Logs pour la visibilité.

- Déclencher et analyser un comportement suspect (ex : téléchargement de fichier de test EICAR via PowerShell).

#### Vérifier l’intégration des logs dans ELK

> Dans l’interface Kibana :

- Aller dans :

`Management > Integrations > Windows`

- Vérifier que les sources suivantes sont **actives** :

  - PowerShell

  - PowerShell Operational

  - Sysmon Operational

  - Windows Defender

> Ces journaux doivent être collectés par l’Elastic Agent installé sur la machine cible

##### Simulation de l'attaque PowerShell

- Commande PowerShell (à lancer depuis une session administrateur) [Anti Malware Testfile](https://www.eicar.org/download-anti-malware-testfile/)

```sh
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

> Cette commande télécharge un fichier de test antivirus (inoffensif mais déclencheur pour les moteurs de détection)

#### Visualisation dans Kibana (ELK)

> Interface : `Analystics > Discover`

#### Requêtes utiles

```sh
event.code:4104
```

> **4104** : Exécution de script PowerShell (logging par PowerShell Operational).

```sh
winlog.event_data.ScriptBlockText : "*eicar*"
```

> Permet de rechercher dans les blocs de script si la commande téléchargée est identifiable.

```sh
event.code:11
```

> **Sysmon Event ID 11** : Détection de fichier créé, utilisé pour observer la génération du fichier malveillant.

#### Incident Response

**Étapes recommandées** :

- Confirmer l’intention : ce téléchargement PowerShell est-il légitime ?

- Isoler l’hôte si un vrai malware avait été détecté.

- Analyser les autres événements PowerShell (event.code 4103, 4104).

- Créer une règle d’alerte dans Kibana :

  - `Stack Management > Rules`

  - Déclencheur : détection de script contenant eicar, ou téléchargement PowerShell via `Invoke-WebRequest`.
