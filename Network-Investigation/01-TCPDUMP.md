# TCPDUMP

- TCPDUMP is a network traffic analysis tool.

- Key Points:

  - Command-line packet analysis tool

  - Capture and analyzes network traffic

  - Works on most UNIX-like systems

## Key Benefits of TCPDUMP

```sh
| Benefit                              | Description                             |
| ------------------------------------ | --------------------------------------- |
| Real-time network monitoring         | See live network traffic as it happens  |
| Filter traffic for targeted analysis | Use filters for precise packet capture  |
| Security investigation & forensics   | Analyze suspicious or malicious traffic |
| Identify traffic-related issues      | Detect misconfigurations or congestion  |
```

## Installing TCPDUMP

```sh
sudo apt install tcpdump
tcpdump --version
```

## Basic Commands

```sh
sudo tcpdump -v                          # Basic verbosity
sudo tcpdump -vv -c 10                   # Capture 10 packets with extra details
sudo tcpdump -D                          # List all interfaces
sudo tcpdump -i any > file.out           # Save output to file (any interface)
sudo tcpdump -i enp0s3 | tee file.out   # Save and view output simultaneously
```

## Capture Packets by Protocols

```sh
ping <IP>                                # ICMP (ping) test
sudo tcpdump -i enp0s3 icmp              # Capture ICMP traffic

sudo tcpdump -i enp0s3 tcp               # Capture TCP traffic
sudo tcpdump -i enp0s3 udp               # Capture UDP traffic

# Trigger scans (to generate traffic)
sudo nmap -sS <IP>                       # TCP SYN scan
sudo nmap -sU <IP>                       # UDP scan
```

## Capture Packets by Port / Host

```sh
sudo tcpdump -i enp0s3 host <IP>         # All traffic to/from specific IP
sudo tcpdump -i enp0s3 host <IP> -vv     # Same with verbose output

# Trigger scan to test
sudo nmap -sS <IP>
```

## Filter by Source and Destination

```sh
# Specific source and destination IP
sudo tcpdump -i enp0s3 src host <SRC_IP> and dst host <DST_IP>

# Resolve domain name to IP
nslookup example.be

# Filter: source IP to destination port 443 (HTTPS)
sudo tcpdump -i enp0s3 src host <SRC_IP> and dst port 443
sudo tcpdump -i enp0s3 src host <SRC_IP> and dst port 443 -vvv

# Filter: source IP to destination subnet
sudo tcpdump -i enp0s3 src host <SRC_IP> and dst net 192.168.1.0/24
```

## Detect Network Scanning (e.g. Nmap SYN scan)

```sh
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack = 0'
# Captures SYN packets without ACK — typical of stealth/Nmap scans

# Launch a scan to test this
sudo nmap -sS <IP>
```
