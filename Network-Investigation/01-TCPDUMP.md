# TCPDUMP – Analyse du trafic réseau

- `tcpdump` est un outil en ligne de commande puissant permettant de capturer, surveiller et analyser le trafic réseau sur une interface.

#### Avantages de TCPDUMP

```sh
| Fonctionnalité                 | Description                                     |
| ------------------------------ | ----------------------------------------------- |
| Surveillance en temps réel     | Capture en direct du trafic réseau              |
| Filtres puissants              | Analyse ciblée via expressions précises         |
| Utilisation en criminalistique | Enquête sur les incidents de sécurité           |
| Diagnostic réseau              | Identification de pannes ou d'anomalies         |
| Sauvegarde des paquets         | Écriture des captures dans des fichiers `.pcap` |
```

#### Installation et version

```sh
sudo apt install tcpdump
tcpdump --version
```

#### Structure de base de la commande

```sh
tcpdump [options] [expression]
```

- **[Options fréquentes]**

```sh
| Option              | Description                                                |
| ------------------- | ---------------------------------------------------------- |
| `-i [interface]`    | Interface réseau à surveiller (ex. `eth0`, `wlan0`, `any`) |
| `-c [nombre]`       | Nombre de paquets à capturer                               |
| `-w [fichier]`      | Sauvegarde la capture dans un fichier `.pcap`              |
| `-r [fichier]`      | Lit une capture précédemment enregistrée                   |
| `-n`                | Ne pas résoudre les noms DNS                               |
| `-v`, `-vv`, `-vvv` | Niveau de verbosité croissant (plus de détails)            |
| `-s [snaplen]`      | Nombre d’octets à capturer par paquet                      |
```

- **[Expressions de filtrage]**

```sh
| Expression           | Description                                         |
| -------------------- | --------------------------------------------------- |
| `host [IP]`          | Trafic provenant ou destiné à une IP                |
| `port [num]`         | Trafic sur un port spécifique                       |
| `src [IP]`           | Paquets provenant d’une IP source                   |
| `dst [IP]`           | Paquets destinés à une IP cible                     |
| `tcp`, `udp`, `icmp` | Filtre par protocole                                |
| `and`, `or`, `not`   | Opérateurs logiques combinés pour filtrage complexe |
```

#### Commandes de base utiles

```sh
sudo tcpdump -v                         # Capture avec détails
sudo tcpdump -vv -c 10                  # Capture les 10 premiers paquets (détails ++)
sudo tcpdump -D                         # Liste des interfaces disponibles
sudo tcpdump -i any                     # Capture sur toutes les interfaces
sudo tcpdump -i any > file.out          # Enregistre dans un fichier texte
sudo tcpdump -i wlp3s0 | tee file1.out  # Capture affichée et enregistrée simultanément
```

#### Captures par protocole ICMP (Ping)

```sh
ping <IP>
sudo tcpdump -i wlp3s0 icmp
```

#### TCP

```sh
sudo tcpdump -i wlp3s0 tcp
```

#### UDP

```sh
sudo tcpdump -i wlp3s0 udp
```

## Captures par ports et IP

#### Trafic vers ou depuis une IP

```sh
sudo tcpdump -i wlp3s0 host <IP>
```

#### Source + Port

```sh
sudo tcpdump -i wlp3s0 src host <IP> and dst port 443
```

#### Source + Destination réseau

```sh
sudo tcpdump -i wlp3s0 src host <IP> and dst net 192.168.1.0/24
```

## Détection de scans SYN (SYN scan)

#### Scan SYN (nmap -sS)

```sh
# Filtrer uniquement les paquets SYN (sans ACK)
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and tcp[tcpflags] & (tcp-ack) == 0'
```

#### Scan SYN sur les ports 80 et 443 + capture vers fichier

```sh
tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and port (443 or 80)' -w scan_probes.pcap
```

#### commande d'attaque avec nmap

```sh
sudo nmap -sS <IP>   # Scan TCP SYN
sudo nmap -sU <IP>   # Scan UDP
```

#### analyser le contenu ASCII (HTTP, login/password)

```sh
sudo tcpdump -i eth0 -A | grep "password"
```
