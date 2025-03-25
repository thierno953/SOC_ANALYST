# TCPDUMP

#### Qu'est-ce que TCPDUMP ?

- TCPDUMP est un outil d'analyse du trafic réseau.
- Points clés:
  - Outil d'analyse de paquets en ligne de commande
  - Capture et analyse le trafic réseau
  - Fonctionne sur la plupart des systèmes de type UNIX

#### Principaux avantages de TCPDUMP

```sh
|------------------------------------------|-----------------------------------------------|
| Surveillance du réseau en temps réel     | Filtrer le trafic pour une analyse spécifique |
|------------------------------------------|-----------------------------------------------|
| Enquête de sécurité et criminalistique   | Identifier les problèmes de trafic            |
|------------------------------------------|-----------------------------------------------|
```

#### Configuration de TCPDUMP

```sh
sudo apt install tcpdump
tcpdump --version
```

#### Format de la commande TCPDUMP

- Format de commande TCPDUMP de base

```sh
tcpdump [options] [expression]
```

- **[Options]**

```sh
-i [interface]  : Spécifie l'interface réseau (ex. 'eth0', 'wlan0').
-c [count]      : Capture un nombre spécifique de paquets (ex. '-c 100' pour 100 paquets).
-w [file]       : Écrit les paquets capturés dans un fichier (ex. '-w capture.pcap').
-r [file]       : Lit un fichier de capture existant (ex. '-r capture.pcap').
-n              : Désactive la résolution DNS pour afficher les adresses IP brutes.
-v, -vv, -vvv   : Augmente le niveau de verbosité pour des détails supplémentaires.
-s [snaplen]    : Définit la longueur du snapshot (nombre d'octets capturés par paquet).
```

- **[Expression]**

```sh
host [IP]       : Capture le trafic provenant ou destiné à une adresse IP spécifique.
port [number]   : Capture le trafic provenant ou destiné à un port spécifique.
src [IP]        : Filtre les paquets provenant d'une adresse IP source.
dst [IP]        : Filtre les paquets destinés à une adresse IP cible.
tcp, udp, icmp  : Capture uniquement un protocole spécifique.
```

#### Commandes de base

```sh
sudo tcpdump -v
sudo tcpdump -vv -c 10
sudo tcpdump -D
sudo tcpdump -i any
sudo tcpdump -i any > file.out
sudo tcpdump -i wlp3s0 | tee file1.out
```

#### Capturer des paquets par protocoles

```sh
ping <IP>
sudo tcpdump -i wlp3s0 icmp
sudo tcpdump -i wlp3s0 tcp

# Attack
sudo nmap -sS <IP>
sudo nmap -sU <IP>
||
sudo tcpdump -i wlp3s0 udp
```

#### Capturer des paquets à l'aide de ports

```sh
sudo tcpdump -i wlp3s0 host <IP>

# Attack
sudo nmap -sS <IP>

||
sudo tcpdump -i wlp3s0 host <IP> -vv

# Attack
sudo nmap -sS <IP>
```

#### Capturer des paquets par source et destination spécifiques

```sh
sudo tcpdump -i wlp3s0 src host <IP> and dst host <IP>

nslookup cfitech.be

sudo tcpdump -i wlp3s0 src host <IP> and dst port 443
sudo tcpdump -i wlp3s0 src host <IP> and dst port 443 -vvv
sudo tcpdump -i wlp3s0 src host <IP> and dst net 192.168.1.0/24
```

#### Capturer les sondes d'analyse du réseau

```sh
# Une attaque par scan SYN

          SYN (443, 80)
Hacker  --------------------->  Serveur
          SYN + ACK
----------------------------------------
        tcpdump capture active
```

```sh
#tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack = 0'

tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0 and (port 80 or port 443)'

tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and port (443 or 80)' -w scan_probes.pcap

# Attack
sudo nmap -sS <IP>
```
