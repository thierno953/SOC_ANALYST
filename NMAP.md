# Scan simple d'une machine

```sh
nmap scanme.nmap.org
```

# Scan d'un réseau entier (ex. /24 = 254 hôtes)

```sh
nmap 192.168.10.0/24
```

# Scan avec détection avancée des services et OS

```sh
sudo nmap -A scanme.nmap.org
```

# Scan furtif (SYN Scan - souvent non détecté par les IDS)

```sh
sudo nmap -sS scanme.nmap.org
```

# Lister tous les appareils sur le réseau (via cache ARP)

```sh
arp -a
```
