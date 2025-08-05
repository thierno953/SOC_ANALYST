# Basic Host Scan

- Scans a single target to identify open ports and basic services.

```sh
nmap scanme.nmap.org
```

# Scan an Entire Network (/24 = 254 hosts)

- Scans all hosts in the `192.168.10.0/24` subnet.

```sh
nmap 192.168.10.0/24
```

# Advanced Service and OS Detection

- Enables aggressive scan: OS detection, version detection, script scanning, and traceroute.

```sh
sudo nmap -A scanme.nmap.org
```

# Stealth Scan (SYN Scan – often undetected by IDS)

- Performs a stealth SYN scan that does not complete the TCP handshake, making it less likely to be logged by the target.

```sh
sudo nmap -sS scanme.nmap.org
```

# List All Devices on Local Network (via ARP Cache)

- Displays IP and MAC addresses of devices that the local machine has communicated with recently.

```sh
arp -a
```
