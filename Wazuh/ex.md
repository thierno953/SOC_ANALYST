# Wazuh

- Wazuh est une plateforme open source de sécurité combinant XDR (Extended Detection and Response) et SIEM (Security Information and Event Management). Elle offre une surveillance unifiée des endpoints, du cloud et des conteneurs avec des fonctionnalités comme l'analyse de logs, la détection d'intrusions et la conformité réglementaire.

#### HIDS, OSSEC et Wazuh

```sh
----------------------------------------------------------------------
OSSEC créé en        | Fork d'OSSEC         | OSSEC est open source  |
2004 par Daniel CID  | en 2015 pour Wazuh   | Solution HIDS          |
----------------------------------------------------------------------
```

#### HIDS (Host-based Intrusion Detection System)

- Système de détection d'intrusions centré sur les hôtes

- Analyse les activités locales (logs, intégrité fichiers, processus)

- Déploiement obligatoire sur chaque appareil surveillé

#### Différences clés Wazuh/OSSEC

```sh
------------------------------------------------------------------------
Fonctionnalité	          |     OSSEC	     |       Wazuh               |
-----------------------------------------------------------------------|
Dashboard	                |   Basique	     |      Interface complète   |
--------------------------|----------------|---------------------------|
Détection vulnérabilités  |	  Non	         |      Intégrée             |
--------------------------|----------------|---------------------------|
Sécurité  cloud	          |   Limité	     |      Support étendu       |
--------------------------|----------------|---------------------------|
Communauté	              |   Réduite	     |      Large support        |
------------------------------------------------------------------------
```

#### Wazuh ajoute notamment

- Analyse de vulnérabilités logicielles

- Évaluation de conformité (PCI DSS, GDPR)

- Surveillance des infrastructures cloud

##### Architecture modulaire

- **1 - Wazuh Server**

- Analyse les données des agents

- Génère les alertes de sécurité

- Gère les configurations via API REST

- **2 - Wazuh Indexer**

- Moteur de stockage et d'indexation Elasticsearch

- Gère jusqu'à des pétaoctets de données

- **3 - Wazuh Dashboard**

- Interface web de visualisation Kibana

- Tableaux de bord prédéfinis pour la chasse aux menaces

- **4 - Agents Wazuh**

- Léger (≈10MB RAM)

- Multiplateforme (Windows, Linux, macOS)

- Collecte logs + détection temps réel

#### Architecture Wazuh

##### Deux modes de déploiement

- **Single-node**

- 1 serveur + 1 indexeur + 1 dashboard

- Idéal pour petits environnements

- **Multi-node**

- Cluster avec 2 serveurs (master/worker)

- 3 indexeurs + 1 dashboard

- Haute disponibilité et scalabilité

#### Flux de données

```sh
Agents → Serveur Wazuh (analyse) → Indexeur (stockage) → Dashboard (visualisation)
```

- **NB**: Les communications sont sécurisées par TLS et authentification mutuelle

#### Méthodes de déploiement

- Options principales :

- **1 - Docker :**

- Conteneurs préconfigurés

- Commande exemple

```sh
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.5
```

- Certificats auto-signés inclus

- **2 - Cloud**

- AMI AWS prête à l'emploi

- Images OVA pour virtualisation

- **3 - Kubernetes**

- Orchestration de conteneurs

- Scaling automatique

- **Installation manuelle**

- Packages RPM/DEB

- Compilation depuis sources

#### Fonctionnalités clés

```sh
----------------------------------------------------------------------------------------
Catégorie	                               Fonctionnalités                                |
----------------------------------------------------------------------------------------|
Surveillance réseau	     |  Analyse Suricata - Détection attaques brute force           |
-------------------------|--------------------------------------------------------------|
Intégrité système	       |  Monitoring temps réel des fichiers système (FIM)            |
-------------------------|--------------------------------------------------------------|
Sécurité cloud	         |  Audit AWS/Azure/GCP - Détection configs risquées            |
-------------------------|--------------------------------------------------------------|
Conformité	             |  Rapports automatiques PCI DSS, HIPAA, GDPR                  |
-------------------------|--------------------------------------------------------------|
Réponse aux incidents	   |  Integration Slack/Teams - Auto-réparation (ex: blocage IP)  |
-------------------------|--------------------------------------------------------------
```

#### Intégrations

- TI : VirusTotal

- SIEM : Elastic, Splunk

##### Exemple d'intégration VirusTotal

```sh
<integration>
  <name>virustotal</name>
  <api_key>KEY</api_key>
  <alert_format>json</alert_format>
</integration>
```

#### Enrollment des agents, processus général

- Générer une clé d'enregistrement sur le serveur

- Installer l'agent sur l'endpoint

- Configurer l'IP du serveur

#### Ubuntu

```sh
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
```

#### Windows

- MSI silencieux avec paramètres préconfigurés

- GPO pour déploiement massif

#### Règles Wazuh

- Mécanisme de détection :

- **1 - Décodeurs**: Normalisation des logs bruts

- **2 - Règles**:

- 10 000+ règles prédéfinies

- Syntaxe XML personnalisable

- Niveaux de criticité (0-15)

#### Exemple règle brute force SSH

```sh
<rule id="5715" level="10">
  <if_sid>5716</if_sid>
  <match>Failed password</match>
  <description>SSH brute force attempt</description>
</rule>
```

# Labs pratiques

#### Lab 1 - FIM

- Activer le monitoring sur /etc

- Modifier un fichier critique

- Analyser l'alerte dans le dashboard

#### Lab 2 : Détection d'intrusion réseau avec Suricata

- Configuration de Suricata comme NIDS (Network IDS)

- Intégration avec Wazuh via les logs Suricata (format EVE JSON)

- Scénario type :
  - Générer du trafic suspect (ex: scan de ports depuis la machine attaquant)
  - Analyser les alertes dans le dashboard Wazuh

#### Lab 3 : Détection de vulnérabilités

- Activer le module sur agent/serveur ()

- Lancer un scan manuel via l'API Wazuh

- Visualiser les vulnérabilités (ex: CVE sur Thunderbird) dans Security -> Vulnerability Detector

#### Lab 4 : Détection de commandes malveillantes

- Activer la surveillance

- Déclencher une alerte en exécutant des commandes critiques (rm -rf /, chmod 777, etc.)

- Analyser l'alerte via les règles personnalisées (ex: id: 100100) dans le dashboard

#### Lab 5 - Brute force

- Lancer hydra contre un serveur SSH

- Voir le blocage automatique via active response

- Vérifier l'IP bannie dans /etc/hosts.deny

#### Lab 6 - VirusTotal

- Télécharger un fichier suspect

- Vérifier l'alerte de détection

- Consulter le rapport VT dans l'alerte
