# Suricata

- **IDS/IPS**:

  - Les systèmes IDS (Intrusion Detection System) analysent le trafic réseau pour détecter des signatures ou des comportements anormaux, mais n'interviennent pas directement sur le trafic. En revanche,
    les IPS (Intrusion Prevention System) peuvent bloquer activement les paquets malveillants en fonction des règles définies.
  - Suricata est un moteur IDS/IPS open source qui prend en charge la détection en temps réel et l'inspection du trafic réseau à l'aide de règles basées sur des signatures ou des comportements.

- **Détection automatique du protocole**:

  - Suricata offre une fonctionnalité avancée de détection automatique des protocoles (HTTP, HTTPS, FTP, SMB, etc.), même si ces derniers utilisent des ports non standards. Cela permet d'appliquer
    une logique de détection et de journalisation adaptée à chaque protocole.
  - Cette capacité est utile pour détecter des menaces telles que les communications malveillantes ou les canaux de commande et contrôle.

- **Script Lua**:

  - Suricata prend en charge le scripting Lua pour créer des règles avancées permettant de détecter et de décoder des trafics malveillants complexes qui ne peuvent pas être identifiés par des règles classiques. Cela inclut l'analyse approfondie du trafic pour identifier certains types de malwares.
  - Lua permet également d'étendre les fonctionnalités de Suricata, comme l'ajout de capacités spécifiques pour décoder ou analyser certains protocoles ou formats.

- **Multithreading**:
  - Suricata utilise intensivement le multithreading pour optimiser les performances sur des processeurs multicœurs. Chaque thread peut être configuré pour traiter un ensemble spécifique de tâches, comme l'analyse des paquets ou la gestion des alertes.
  - La configuration du ratio "detect_thread_ratio" permet d'ajuster le nombre de threads en fonction du matériel disponible, ce qui est crucial pour maintenir une performance élevée dans les environnements à fort trafic réseau.

#### Comment fonctionne Suricata ?

- Acquisition de trafic réseau
- Persing et analyse de paquets
- Correspondance de signature
- Inspection approfondie des paquets (en option)
- Action et journalisation

```sh
Rules --> Signature --> Alert --> log (Suricata.log)
```

#### Quels protocoles sont utilisés chez Suricata ?

- Protocoles de base:

  - TCP (protocole de contrôle de transmission)
  - UDP (protocole de datagramme utilisateur)
  - Protocole ICMP (Internet Control Message Protocol)
  - IP (Protocole Internet)

- Protocoles de couche d'application (couche 7):

  - HTTP (protocole de transfert hypertexte)
  - HTTP/2
  - FTP (protocole de transfert de fichiers)
  - TLS/SSL (couche de sécurité de la couche de transport/couche de sockets sécurisée)
  - SMB (bloc de messages du serveur)
  - DNS (Système de noms de domaine)

#### Règles de Suricata

```sh
alert http $HOME_NET any -> $EXTERNAL_NET any (msg: "HTTP GET Request Containing Rule in URI"; flow:established, to_server; http.method; content: "GET"; http.uri; content: "rule"; fast_pattern; classtype:bad-unknown; sid:123; rev:1;)
```

- Une règle/signature se compose des éléments suivants:

  - **Action**, déterminant ce qui se passe lorsque la règle correspond.
  - **Header**, définissant le protocole, les adresses IP, les ports et la direction de la règle.
  - **Rule options**, définissant les spécificités de la règle.

**Rouge** est l'action, **Vert** est l'en-tête et **bleu** sont les options.

- Les actions valides sont:

  - **alert** - générer une alerte (IDS)
  - **pass** - arrêter toute inspection supplémentaire du paquet
  - **drop** - déposer un paquet et générer une alerte
  - **reject** - envoyer une erreur de non-atteinte RST/ICMP à l'expéditeur du paquet correspondant (IPS)
  - **rejectsrc** - identique à simplement rejeter.
  - **rejectdst** - envoie un paquet d'erreur RST/ICMP au récepteur du paquet correspondant
  - **rejectboth** - envoie des paquets d'erreur RST/ICMP aux deux côtés de la conversation

```sh
HOME_NET = <UBUNTU IP>
EXTERNAL_NET = "any"
http = http://google.com/image/rule
```

#### Installation de Suricata IDS sur Ubuntu Server

```sh
uname -a
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
sudo systemctl status suricata
cd /etc/suricata
ls
mkdir rules

cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz && sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules

ls
cd /etc/suricata/rules
```

#### Configuration de Suricata IDS

root@forwarder:~# sudo nano /etc/suricata/suricata.yaml

```sh
vars:
  address-groups:
    HOME_NET: "[IP]"

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

```sh
sudo systemctl restart suricata
sudo systemctl status suricata
```

#### Suricata Logs

```sh
cd /var/log/suricata/
ls
tail -f fast.log
```

#### Labo: Détection de scan Nmap à l'aide de Suricata IDS

```sh
tail -f /var/log/suricata/fast.log
```

#### Attaque

```sh
nmap -sS <IP>

# nmap -sI zombie_host_ip <target IP address>
nmap -sI 149.6.6.6 <target IP address>
```
