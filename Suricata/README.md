# Suricata IDS

- Intrusion Detection Systems (IDS) analyze network traffic to detect known signatures or abnormal behavior. They do not actively intervene with the traffic — unlike IPS (Intrusion Prevention Systems), which can block or reject packets in real time.

#### Example of a Suricata Rule

```sh
alert   http      $HOME_NET  any   ->    $EXTERNAL_NET     any (msg: "Requests"; sid:123; rev:1;)
|        |         |          |             |                |                    |
Action  Protocol  Source    Source_port  Destination  Destination_port   Rule_options
```

## Rule Components:

- **Action**: What Suricata should do when the rule matches (`alert`, `drop`)

- **Header**: Protocol, IPs, ports, and direction

- **Options**: Rule conditions like message (`msg`), signature ID (`sid`), revision (`rev`)

## Valid actions:

- `alert` - Generate an alert (IDS mode)

- `pass` - Skip this packet

- `drop` - Drop the packet and log an alert (IPS mode)

- `reject` - Drop and send TCP RST / ICMP error message

- `rejectsrc` - Error to the sender only

- `rejectdst` - Error to the receiver only

- `rejectboth` - Error to both sender and receiver

## Variable Example

```sh
HOME_NET = [192.168.0.0/24]
EXTERNAL_NET = !$HOME_NET
```

## Installing Suricata IDS on Ubuntu Server

- Official Resources

  - Suricata Quick Start: [https://docs.suricata.io/en/latest/quickstart.html](https://docs.suricata.io/en/latest/quickstart.html)
  - IPS Inline Setup Guide: [https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html](https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html)

## Installation on Ubuntu Server

#### Add Suricata Repository and Install

```sh
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq
```

## Verify Installation

```sh
sudo suricata --build-info
sudo systemctl status suricata
which suricata
whereis suricata
```

## Suricata Configuration

#### Backup and Edit Config

```sh
cd /etc/suricata
sudo cp suricata.yaml suricata.yaml.backup
sudo nano suricata.yaml
```

## Sample Configuration (`suricata.yaml`)

```sh
vars:
  address-groups:
    HOME_NET: "[192.168.0.0/24]"
    EXTERNAL_NET: "!$HOME_NET"

default-rule-path: /var/lib/suricata/rules

rule-files:
  - custom.rules

stats:
  enabled: yes

af-packet:
  - interface: enp0s8
```

## Create Custom Rules

```sh
sudo mkdir -p /var/lib/suricata/rules
sudo touch /var/lib/suricata/rules/custom.rules
```

## Example Rule: ICMP Alert

```sh
sudo nano /var/lib/suricata/rules/custom.rules
```

```sh
# Alert on any ICMP traffic incoming to HOME_NET
alert icmp any any -> $HOME_NET any (msg: "Incoming ICMP packets!"; sid:123; rev:1;)
```

## Start Suricata in IDS Mode

```sh
sudo suricata -c /etc/suricata/suricata.yaml -i enp0s8
```

## View Logs

```sh
cd /var/log/suricata/
tail -f fast.log
```

## Detect Nmap Scan

```sh
tail -f /var/log/suricata/fast.log
nmap -sS <TARGET_IP>
```

## Configure IPS (NFQUEUE)

#### Edit `suricata.yaml` for NFQUEUE

```sh
nfq:
  mode: accept
  repeat-mark: 1
  repeat-mask: 1
  route-queue: 2

af-packet:
  - interface: enp0s8
```

## Add Drop Rule

```sh
sudo nano /var/lib/suricata/rules/custom.rules
```

```sh
# Drop ICMP traffic to HOME_NET
drop icmp any any -> $HOME_NET any (msg: "Drop ICMP packets!"; sid:124; rev:1;)
```

## Configure IPTables for NFQUEUE

```sh
sudo iptables -I INPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -I OUTPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -vnL
```

## Start Suricata in IPS Mode

```sh
sudo suricata -c /etc/suricata/suricata.yaml -q 2
```

## Test Drop Rule

```sh
ping <Suricata_IP>
tail -f /var/log/suricata/fast.log
```

## Remove IPTables Rules (Cleanup)

```sh
sudo iptables -D INPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -D OUTPUT -p icmp -j NFQUEUE --queue-num 2
```

## Update Rules with `suricata-update`

```sh
sudo apt install suricata-update
sudo suricata-update
```

## View Detailed JSON Logs

```sh
tail -f /var/log/suricata/eve.json | jq .
```

## HTTP GET Detection Rule

```sh
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"HTTP GET Detected"; flow:established,to_server; http.method; content:"GET"; sid:125; rev:1;)
```
