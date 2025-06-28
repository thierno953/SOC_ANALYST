# Détection du trafic réseau anormal

- Installer et configurer Suricata IDS

- Intégrer les logs dans Splunk

- Simuler un scan malveillant

- Visualiser et analyser les alertes

- Réagir à l’incident

#### Installation et configuration de Suricata

- [Proofpoint Emerging Threats Rules](https://rules.emergingthreats.net/)

```sh
apt-get update
add-apt-repository ppa:oisf/suricata-stable -y
apt-get install suricata -y
```

#### Téléchargement des règles ET (Emerging Threats) :

```sh
cd /etc/suricata && mkdir rules && cd && cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-7.0.3/emerging.rules.tar.gz
tar -xvzf emerging.rules.tar.gz
mv rules/*.rules /etc/suricata/rules/
chmod 640 /etc/suricata/rules/*.rules
```

#### Exemple d’édition de règle spécifique

```sh
nano /etc/suricata/rules/emerging-malware.rules
```

#### Configuration de Suricata (/etc/suricata/suricata.yaml)

```sh
vars:
  address-groups:
    HOME_NET: "[IP_ADDRESS]"
    EXTERNAL_NET: "any"

stats:
  enabled: yes

default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"

af-packet:
  - interface: enp0s3
```

- **NB** : Remplace [`IP_ADDRESS`] par l’adresse IP réelle de la machine cible (victime)

#### Démarrage et test de Suricata

```sh
systemctl restart suricata
systemctl status suricata
cd /var/log/suricata/
tail -f fast.log
```

## Configuration de Splunk Universal Forwarder

#### Modifier inputs.conf

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

#### Contenu du fichier

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

#### Redémarrer le Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart 
```

## Simulation d’une attaque (Scan TCP SYN)

#### Depuis la machine attacker

```sh
nmap -sS <victim_IP>
```

## Visualisation dans Splunk (Search & Reporting)

> `Requêtes dans Search & Reporting`

```sh
index="network_security_logs" sourcetype="suricata"
index="network_security_logs" sourcetype="suricata" src_ip="<IP_ATTACK_MACHINE>"
```
