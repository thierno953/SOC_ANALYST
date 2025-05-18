# Wazuh

- **Wazuh** = plateforme open source XDR + SIEM

- Surveillance unifiée des endpoints, cloud, conteneurs

- `Fonctionnalités` : analyse logs, détection d'intrusions, conformité

#### HIDS, OSSEC, Wazuh

```sh
----------------------------------------------------------
OSSEC créé en        | Fork d'OSSEC        | Type         |
2004 (Daniel Cid)    | en 2015 → Wazuh     | HIDS (Open)  |
----------------------------------------------------------
```

#### HIDS (Host-based Intrusion Detection System)

- Détection d'intrusion sur hôtes (pas réseau)

- Analyse fichiers, processus, logs

- Déploiement agent sur chaque machine

#### Wazuh vs OSSEC

```sh
| Fonctionnalité            | OSSEC       | Wazuh                       |
|---------------------------|-------------|-----------------------------|
| Dashboard                 | Basique     | Interface complète          |
| Détection vulnérabilités  | Non         | Oui                         |
| Sécurité cloud            | Limité      | Support étendu              |
| Communauté                | Petite      | Active et large             |
```

#### Wazuh ajoute

- Détection vulnérabilités softs

- Conformité (PCI-DSS, GDPR, etc.)

- `Audit cloud` : AWS, Azure, GCP

#### Architecture Wazuh

- **Composants**

  - **Wazuh Server** → analyse, alertes, API

  - **Wazuh Indexer** → basé sur Elasticsearch

  - **Dashboard** → visualisation (Kibana-like)

  - **Agents** → collectent logs (Windows, Linux, macOS)

- **Modes déploiement**

  - **Single-node** : simple, tout-en-un

  - **Multi-node** : scalabilité, HA, clusters

#### Flux de données

`Agents → Server (analyse) → Indexer (stockage) → Dashboard (visualisation)`

- **NB**: Les communications sont sécurisées par TLS et authentification mutuelle

#### Déploiement

- **1 - Docker**

```sh
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.5
```

- Conteneurs préconfigurés

- Certificats auto-signés

- **2 - Cloud**

  - AMI AWS, images OVA

- **3 - Kubernetes**

  - Scalabilité auto, orchestration

- **Installation manuelle**

  - Packages DEB/RPM, ou compilation

#### Fonctionnalités

```sh
| Catégorie                | Fonctionnalités                        |
|--------------------------|----------------------------------------|
| Surveillance réseau      | Suricata, brute force, scan ports      |
| Intégrité fichiers (FIM) | En temps réel                          |
```

#### Intégrations

- **VirusTotal**

- **SIEM** : Elastic, Splunk

```sh
<integration>
  <name>virustotal</name>
  <api_key>KEY</api_key>
  <alert_format>json</alert_format>
</integration>
```

#### Enrollment Agent

- Générer clé sur serveur
- Installer agent → config IP serveur

#### Ubuntu

```sh
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
```

#### Windows

- MSI silencieux + déploiement GPO

#### Détection = Décodeurs + Règles

- 10 000+ règles prédéfinies

- Syntaxe XML personnalisée

- Niveaux 0 à 15

#### Exemple règle SSH brute force

```sh
<rule id="5715" level="10">
  <if_sid>5716</if_sid>
  <match>Failed password</match>
  <description>SSH brute force attempt</description>
</rule>
```

### LABS: Déploiement d’une solution XDR/SIEM avec Wazuh et Docker dans un environnement Linux/Windows

#### Lab 1 - FIM

- Activer /etc, modifier fichier

- Voir alerte dans dashboard

#### Lab 2 - Suricata IDS

- Installer + activer Suricata (format JSON EVE)

- Scanner avec nmap, alerte → dashboard

#### Lab 3 - Vulnérabilités

- Activer `Vulnerability Detector`

- Scanner → visualiser dans `Security > Vulnerability`

#### Lab 4 - Commandes malveillantes

- Exécuter `rm -rf /`, `chmod 777`

- Analyse via règles personnalisées

#### Lab 5 - SSH Brute Force

- Hydra → serveur SSH

- Blocage auto → check `/etc/hosts.deny`

#### Lab 6 - VirusTotal

- Télécharger fichier douteux

- Voir analyse VT dans l’alerte

### Installation Wazuh avec Docker

- [Configuration système de base](https://documentation.wazuh.com/4.7/deployment-options/docker/docker-installation.html)

```sh
sudo -i
sysctl -w vm.max_map_count=262144
sysctl -p
```

- Docker

```sh
curl -sSL https://get.docker.com/ | sh
usermod -aG docker $USER
systemctl start docker
systemctl status docker
```

- Docker Compose

```sh
curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

- Clonage + Lancement [Clonage du dépôt Wazuh](https://documentation.wazuh.com/4.7/deployment-options/docker/wazuh-container.html)

```sh
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.5
cd wazuh-docker/single-node/

docker-compose -f generate-indexer-certs.yml run --rm generator
docker-compose up -d
```

- Configuration du firewall

```sh
sudo ufw enable
sudo ufw allow 9200/tcp
```

#### Installation AGENT

- Script d'installation de l'agent linux

![agent](/Wazuh/assets/01.png)

![agent](/Wazuh/assets/02.png)

- Script d'installation de l'agent windows : `C:\Program Files > ossec-agent > win32ui`

- [CONFIGURATIONS SPÉCIFIQUES](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)

- **Lab #1:** File Integrity Monitoring (FIM)

  - Configuration FIM - Manager

```sh
docker ps

docker exec -it single-node-wazuh.manager-1 bash

/var/ossec/bin/wazuh-control restart

apt update && apt install -y nano
```

```sh
nano /var/ossec/etc/ossec.conf
```

- Modification ceci dans le fichier de configuration de Wazuh Manager

```sh
<ossec_config>
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>yes</logall>
    <logall_json>yes</logall_json>
  </global>
</ossec_config>
```

```sh
service wazuh-manager restart
```

- Configuration Agent

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Agent

```sh
<syscheck>
  <directories check_all="yes" realtime="yes" report_changes="yes">/root</directories>
  <frequency>3600</frequency>
  <scan_on_start>yes</scan_on_start>
</syscheck>
```

```sh
systemctl restart wazuh-agent
touch samplefile.txt
```

- Dans l'interface Wazuh : `Modules > agent01 > Security events`

- **Lab #2:** Detecting Network Instruction Using Suricata IDS

  - Installation Suricata

```sh
add-apt-repository ppa:oisf/suricata-stable -y
apt-get update
apt-get install suricata -y

cd /etc/suricata && mkdir rules && cd

cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
tar -xvzf emerging.rules.tar.gz
mv rules/*.rules /etc/suricata/rules/
chmod 640 /etc/suricata/rules/*.rules

cd /etc/suricata/rules/
```

- Configuration Suricata

```sh
nano /etc/suricata/suricata.yaml
```

```sh
vars:
  address-groups:
    HOME_NET: "[192.168.129.58]"
    EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"

stats:
  enabled: yes

af-packet:
  - interface: enp0s3
    threads: auto               # Utilise tous les CPU dispo
    cluster-id: 99              # Identifiant de groupe de capture
    cluster-type: cluster_flow  # Classe les flux réseau pour mieux gérer
    defrag: yes                 # Réassemble les paquets fragmentés

outputs:
  - eve-log:
      enabled: yes
      filetype: regular
      filename: /var/log/suricata/eve.json
```

- Configuration Wazuh pour Suricata

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Manager

```sh
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

```sh
sudo systemctl restart suricata
sudo systemctl restart wazuh-agent
```

```sh
cd /var/log/suricata/
tail -f fast.log
```

- Simulation d'attaque et visualise les événements

```sh
# Exécutez
root@attack:~# nmap -sS <IP_VICTIME>
root@attack:~# nmap -sS -T4 -Pn <IP_VICTIME>
root@attack:~# nmap -p 22 --script ssh-brute <IP_VICTIME>
```

- Dans l'interface Wazuh : `Modules > agent01 > Security events > rule.id`

- **Lab #3:** Détection de vulnérabilités

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Manager

```sh
<vulnerability-detection>
  <enabled>yes</enabled>
  <interval>10m</interval>
  <min_full_scan_interval>12h</min_full_scan_interval>
  <run_on_start>yes</run_on_start>
  <alert_if_above>high</alert_if_above>

  <!-- Ubuntu -->
  <provider name="canonical">
    <enabled>yes</enabled>
    <os>focal</os>
    <os>jammy</os>
    <exclude>libreoffice*</exclude>
  </provider>

  <!-- Windows -->
  <provider name="msu">
    <enabled>yes</enabled>
    <update_interval>1h</update_interval>
  </provider>

  <!-- Red Hat (optionnel) -->
  <provider name="redhat">
    <enabled>yes</enabled>
    <os>8</os>
    <os>9</os>
  </provider>
</vulnerability-detection>
```

```sh
service wazuh-manager restart
service wazuh-manager status
```

- Pour vérification que ça fonctionne Sur le Manager

```sh
tail -f /var/ossec/logs/alerts/alerts.log | grep "vulnerability"
```

- Pour vérification que ça fonctionne Sur l'agent (Linux)

```sh
sudo wazuh-control info | grep -i "vulnerability"
```

- Dans l'interface Wazuh : `Modules > agent01 > Vulnerabilities`

- **Lab #4:** Auditd

  - Installation Auditd

```sh
apt install auditd -y
systemctl enable --now auditd
```

- Configuration Auditd

```sh
cd /var/log/audit/ && tail -f audit.log
```

```sh
nano /etc/audit/audit.rules
```

- Ajout ces règles dans /etc/audit/audit.rules

```sh
-a exit,always -F euid=0 -F arch=b64 -S execve -k audit-wazuh-c
-a exit,always -F euid=0 -F arch=b32 -S execve -k audit-wazuh-c
```

- Si nécessaire, configurez : **Management > CDB lists > audit-keys**

```sh
auditctl -R /etc/audit/audit.rules
netstat
```

- Configuration Wazuh pour Auditd

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Manager

```sh
<ossec_config>
  <localfile>
    <log_format>audit</log_format>
    <location>/var/log/audit/audit.log</location>
  </localfile>
</ossec_config>
```

```sh
systemctl restart auditd wazuh-agent
```

- Dans l'interface Wazuh : **Modules > agent01 > Security events**

- **Lab #5:** Detecting and Blocking SSH brute force attacks

  - Configuration Manager pour SSH brute force

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Manager

```sh
<ossec_config>
  <!--Active Response-->
  <active-response>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5763</rules_id>
    <timeout>180</timeout>
  </active-response>
</ossec_config>
```

```sh
service wazuh-manager restart
```

- Modifiez : **Management > Rules > Manage rules files > 0095-sshd_rules.xml**

- Configuration Agent pour les réponses actives

```sh
root@wazuh:/# cd /var/ossec/active-response/bin/
root@wazuh:/var/ossec/active-response/bin# ls
default-firewall-drop  firewalld-drop  ipfw          npf            restart.sh
disable-account        host-deny       kaspersky     pf             route-null
firewall-drop          ip-customblock  kaspersky.py  restart-wazuh  wazuh-slack
root@wazuh:/var/ossec/active-response/bin#
```

- Ajout ceci dans le fichier de configuration de Wazuh Agent

![agent](/Wazuh/assets/03.png)

```sh
nano /var/ossec/etc/ossec.conf
```

```sh
<ossec_config>
  <!-- Cluster Configuration (Optional) -->
  <cluster>
    <name>wazuh</name>
    <node_name>node01</node_name>
    <node_type>master</node_type>
    <key>aa093264ef885029653eea20dfcf51ae</key>
    <port>1516</port>
    <bind_addr>0.0.0.0</bind_addr>
    <nodes>
        <node>wazuh.manager</node>
    </nodes>
    <hidden>no</hidden>
    <disabled>yes</disabled>
  </cluster>

  <!-- VirusTotal Integration -->
  <integration>
    <name>virustotal</name>
    <api_key>YOUR_VIRUSTOTAL_API_KEY</api_key> <!-- Replace with your API key -->
    <rule_id>100200,100201</rule_id>
    <alert_format>json</alert_format>
  </integration>

  <!-- Active Response Logging -->
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>
</ossec_config>
```

- N'oubliez pas de remplacer YOUR_VIRUSTOTAL_API_KEY par votre clé API VirusTotal.

```sh
service wazuh-manager restart
```

- Dans l'interface Wazuh : `Modules > agent01 > Security events`

#### Pour tester l'attaque par force brute SSH

```sh
hydra -t 4 -l root pass.txt <IP_VICTIME> ssh
```

- **Lab #6:** Detecting Malicious files using virustotal

  - Configuration FIM

```sh
nano /var/ossec/etc/ossec.conf
```

- Ajout ceci dans le fichier de configuration de Wazuh Agent

```sh
# <!-- File integrity monitoring -->
<syscheck>
  <!-- Surveillance renforcée -->
  <directories check_all="yes" realtime="yes" report_changes="yes">/root</directories>
  <directories check_all="yes" realtime="yes" report_changes="yes">/home/user1</directories>

  <!-- Surveillance standard pour /etc (sans report_changes global) -->
  <directories check_all="yes" realtime="yes">/etc</directories>
  <directories check_all="yes" realtime="yes" report_changes="yes">/etc/shadow</directories> <!-- Fichier spécifique -->

  <!-- Surveillance légère pour les binaires système -->
  <directories check_sum="yes" check_owner="yes" check_group="yes">/usr/bin,/usr/sbin</directories>

  <!-- Exclusions -->
  <ignore>/etc/adjtime</ignore>
  <ignore>/etc/resolv.conf</ignore>
  <ignore>/usr/bin/*.log</ignore>
  <ignore type="sregex">\.swp$|\.tmp$</ignore>  <!-- Fichiers temporaires -->
</syscheck>
```

```sh
systemctl restart wazuh-agent
```

- Modification : `Management > Rules > Manage rules files > Edit local_rules.xml`

```sh
<!-- Local rules -->

<!-- Modify it at your will. -->
<!-- Copyright (C) 2015, Wazuh Inc. -->

<!-- Example -->
<group name="local,syslog,sshd,">

  <!--
  Dec 10 01:02:02 host sshd[1234]: Failed none for root from 1.1.1.1 port 1066 ssh2
  -->
  <rule id="100001" level="5">
    <if_sid>5716</if_sid>
    <srcip>1.1.1.1</srcip>
    <description>sshd: authentication failed from IP 1.1.1.1.</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,</group>
  </rule>

  <rule id="100200" level="7">
    <if_sid>550</if_sid>
    <field name="file">/root</field>
    <description>File modified in /root directory.</description>
  </rule>
  <rule id="100201" level="7">
    <if_sid>554</if_sid>
    <field name="file">/root</field>
    <description>File added to /root directory.</description>
  </rule>

</group>
```

- Téléchargement un fichier de test [EICAR](https://www.eicar.org/)

```sh
root@agent:~# curl -Lo /root/eicar.com https://secure.eicar.org/eicar.com && sudo ls -lah /root/eicar.com
```

- Vérification les logs dans l'interface Wazuh : `Modules > Security events`
