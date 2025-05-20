# TCPDUMP

- TCPDUMP est un outil CLI d'analyse de paquets réseau, disponible sur les systèmes UNIX.

- Fonctionnalités clés pour :

  - Capturer et analyser le trafic réseau en temps réel
  - Diagnostiquer des problèmes de connectivité
  - Détecter des activités suspectes (scans, attaques DDoS)

- Compatibilité : Linux, macOS, BSD (nécessite les droits root)

#### Principaux avantages de TCPDUMP

```sh
|------------------------------------------|-----------------------------------------------|
| Surveillance du réseau en temps réel     | Filtrer le trafic pour une analyse spécifique |
|------------------------------------------|-----------------------------------------------|
| Enquête de sécurité et criminalistique   | Identifier les problèmes de trafic            |
|------------------------------------------|-----------------------------------------------|
```

#### Installation

- Sur Debian/Ubuntu

```sh
sudo apt update && sudo apt install tcpdump -y
```

- Vérification

```sh
tcpdump --version
```

#### Syntaxe de Base

```sh
tcpdump [options] [filtres]
```

- **Options Principales**

```sh
-------------|-------------------------------------------|-------------------------|
Option	           Description	                                  Exemple
-------------|-------------------------------------------|-------------------------|
-i eth0	         Interface réseau (any pour toutes)	        -i wlan0
-------------|-------------------------------------------|-------------------------|
-c 50            Limite le nombre de paquets	              -c 100
-------------|-------------------------------------------|-------------------------|
-w file.pcap	   Sauvegarde dans un fichier .pcap	          -w capture.pcap
-------------|-------------------------------------------|-------------------------|
-r file.pcap	   Lit une capture existante	                -r traffic.pcap
-------------|-------------------------------------------|-------------------------|
-n	             Désactive la résolution DNS	            -nn (ports aussi)
-------------|-------------------------------------------|-------------------------|
-v	             Verbosité (-vv pour plus de détails)	      -vvv
-------------|-------------------------------------------|-------------------------|
-s 0          	 Capture complète des paquets	               -s 1500
-------------|-------------------------------------------|-------------------------|
```

#### Filtres Essentiels

- Par Adresse/Port

```sh
sudo tcpdump host 192.168.1.100           # Trafic vers/depuis l'IP
sudo tcpdump src 192.168.1.1             # Paquets depuis l'IP source
sudo tcpdump dst port 443                # Trafic HTTPS
```

- Par Protocole

```sh
sudo tcpdump icmp                         # Pings (ICMP)
sudo tcpdump tcp                          # TCP uniquement
sudo tcpdump udp port 53                  # DNS
```

- Combinaisons

```sh
sudo tcpdump 'src 192.168.1.100 and dst port 80'  # HTTP depuis une IP
sudo tcpdump 'net 192.168.1.0/24 and not port 22' # Tout sauf SSH
```

#### Cas d'Usage Pratiques

- Capture Basique

```sh
sudo tcpdump -i eth0 -c 10 -vv  # 10 paquets avec détails
```

- Sauvegarde & Analyse

```sh
sudo tcpdump -i any -w traffic.pcap       # Sauvegarde
tcpdump -r traffic.pcap | grep "HTTP"     # Filtre post-capture
```

- Détection de Scans Réseau

```sh
# Scan SYN (détection Nmap)
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0'

# Scan UDP
sudo tcpdump -i eth0 'udp and portrange 1-1024'
```

- Analyse HTTP/HTTPS

```sh
sudo tcpdump -A -i eth0 'port 80'         # HTTP en clair
sudo tcpdump -X -i eth0 'port 443'        # HTTPS (en-têtes seulement)
```

#### Bonnes Pratiques

- Limiter l'impact : Utilisez `-c` pour éviter les captures infinies

```sh
sudo tcpdump -i eth0 -c 500 'port 80'  # 500 paquets max
```

- Combiner avec Wireshark

```sh
sudo tcpdump -i eth0 -w capture.pcap && wireshark capture.pcap
```

- Filtres complexes : Utilisez des parenthèses

```sh
sudo tcpdump '(tcp or udp) and (port 80 or port 443)'
```

#### Astuces Avancées

- Analyser le Trafic en Temps Réel

```sh
watch -n 1 "sudo tcpdump -i eth0 -c 5 port 80"  # Aperçu toutes les 1s
```

- Détection d'Attaques

```sh
# SYN Flood
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0' -c 100 | grep "SYN"

# Trafic anormal
sudo tcpdump -i eth0 'port not (22,80,443,53)'
```

- Capture du trafic web depuis une IP spécifique

```sh
sudo tcpdump -i wlan0 'src 192.168.1.100 and (port 80 or port 443)' -w web_traffic.pcap -vv
```
