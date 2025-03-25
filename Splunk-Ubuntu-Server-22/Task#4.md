# Détection de trafic réseau anormal

- **Objectif** : Détecter et analyser le trafic réseau anormal à l'aide de Suricata (IDS) et visualiser les résultats dans Splunk.
- **Étapes** :
  - Installer et configurer Suricata.
  - Configurer Splunk pour surveiller les logs de Suricata.
  - Simuler une attaque et visualiser les résultats.
  - Répondre à l'incident.

#### 1 - Installation et configuration de Suricata

- Mettre à jour les paquets

```sh
root@forwarder:~# sudo apt-get update
```

- Installer les dépendances nécessaires

```sh
root@forwarder:~# sudo apt-get install software-properties-common
```

- Ajouter le référentiel Suricata

```sh
root@forwarder:~# sudo add-apt-repository ppa:oisf/suricata-stable
```

- Mettre à jour les paquets après l'ajout du référentiel

```sh
root@forwarder:~# sudo apt-get update
```

- Installer Suricata

```sh
root@forwarder:~# sudo apt-get install suricata -y
```

- Télécharger et configurer les règles Suricata
  - Télécharger les règles Emerging Threats

#### Lien de référence :

[emergingthreats](https://rules.emergingthreats.net)

```sh
root@forwarder:~# cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
```

- Extraire et déplacer les règles

```sh
root@forwarder:~# sudo tar -xvzf emerging.rules.tar.gz && sudo mkdir /etc/suricata/rules && sudo mv rules/*.rules /etc/suricata/rules/
```

- Définir les permissions des fichiers de règles

```sh
root@forwarder:~# sudo chmod 640 /etc/suricata/rules/*.rules
```

- Vérifier les règles installées

```sh
root@forwarder:~# cd /etc/suricata/rules/
root@forwarder:/etc/suricata/rules/~# ls
```

- Éditer une règle spécifique (optionnel)

```sh
root@forwarder:/etc/suricata/rules/~# nano emerging-malware.rules
```

- Revenir au répertoire racine

```sh
root@forwarder:/etc/suricata/rules/~# cd ../../..
```

- Éditer le fichier de configuration de Suricata

```sh
root@forwarder:~# sudo nano /etc/suricata/suricata.yaml
```

- Coller ou modifier les sections suivantes

```sh
rule-files:
  - emerging-web_client.rules
  - emerging-malware.rules
  - emerging-attack_response.rules
  - emerging-trojan.rules

# ---------------------------

vars:
  address-groups:
    HOME_NET: "[192.168.129.206]"

    #EXTERNAL_NET: "!$HOME_NET"
    EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"

# Global stats configuration
stats:
enabled: yes

# Linux high speed capture support
af-packet:
  - interface: enp0s3
```

- Redémarrer le service Suricata

```sh
root@forwarder:~# sudo systemctl restart suricata
```

- Vérifier le statut de Suricata

```sh
root@forwarder:~# sudo systemctl status suricata
```

- Mettre à jour les règles Suricata

```sh
root@forwarder:~# suricata-update
```

- Surveiller le trafic réseau avec tcpdump (optionnel)

```sh
root@forwarder:~# tcpdump -i enp0s3
```

- Surveiller les logs de Suricata en temps réel

```sh
root@forwarder:~# cd /var/log/suricata/
root@forwarder:/var/log/suricata# tail -f fast.log
```

#### 2 - Configuration de Splunk pour surveiller les logs de Suricata

- Éditer le fichier de configuration des inputs de Splunk

```sh
root@forwarder:~# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Ajouter ou modifier les sections suivantes

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = auditd
index = linux_file_integrity

[monitor:///var/log/suricata/fast.log]
disabled = false
index = network_security_logs
sourcetype = suricata
```

- Redémarrer Splunk pour appliquer les changements

```sh
root@forwarder:/var/log/suricata# /opt/splunkforwarder/bin/splunk restart
```

#### 3 - Simulation d'une attaque et analyse des logs dans Splunk

- Scanner le système cible avec Nmap

```sh
root@attack:~# nmap -sS <IP SPLUNK>
root@attack:~# nmap -A <IP SPLUNK>
```

#### Victime (SPLUNK)

- Surveiller les logs de Suricata en temps réel

```sh
root@forwarder:~# tail -f /var/log/suricata/fast.log
```

- Mettre à jour la configuration de Splunk pour inclure eve.json

```sh
root@forwarder:~# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Ajouter ou modifier la section suivante

```sh
[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

- Redémarrer Splunk

```sh
root@forwarder:~# /opt/splunkforwarder/bin/splunk restart
```

#### Attaquant (suite)

- Lancer un autre scan Nmap

```sh
root@attack:~# nmap -sS <IP SPLUNK>
```

##### Recherche et reporting dans Splunk

```sh
index="network_security_logs" sourcetype="suricata"
index="network_security_logs" sourcetype="suricata" src_ip="<IP Attack>"
```
