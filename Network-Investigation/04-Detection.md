# Suricata IDS

- Intrusion Detection Systems (IDS) analyze network traffic to detect known signatures or abnormal behavior. They do not actively intervene with the traffic — unlike IPS (Intrusion Prevention Systems), which can block or reject packets in real time.

#### Example of a Suricata Rule

```sh
alert   http      $HOME_NET  any   ->    $EXTERNAL_NET     any (msg: "Requests"; sid:123; rev:1;)
|        |         |          |             |                |                    |
Action  Protocol  Source    Source_port  Destination  Destination_port   Rule_options
```

#### A rule/signature consists of:

- **Action**: What Suricata should do if the rule matches.

- **Header**: Defines the protocol, IP addresses, ports, and direction.

- **Rule options**: Specific conditions and metadata (e.g., message, signature ID, revision).

  > **Red** = Action

  > **Green** = Header

  > **Blue** = Rule Options

#### Valid actions:

- `alert` - generate an alert (for IDS use)

- `pass` - stop inspecting the packet further

- `drop` - drop the packet and generate an alert

- `reject` - drop and send a RST/ICMP unreachable message to the sender

- `rejectsrc` - send error to the sender only

- `rejectdst` - send error to the receiver only

- `rejectboth` - send errors to both sender and receiver

#### Variable Example

```sh
HOME_NET = <YOUR_INTERNAL_IP>
EXTERNAL_NET = "any"
```

## Installing Suricata IDS on Ubuntu Server

- Official Resources

  - Suricata Quick Start: [https://docs.suricata.io/en/latest/quickstart.html](https://docs.suricata.io/en/latest/quickstart.html)
  - IPS Inline Setup Guide: [https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html](https://docs.suricata.io/en/latest/setting-up-ipsinline-for-linux.html)

## Installation steps:

```sh
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install suricata jq
```

## Check installation:

```sh
sudo suricata --build-info
sudo systemctl status suricata
which suricata
whereis suricata
```

## Suricata Configuration

- Backup and edit config file:

```sh
cd /etc/suricata
sudo cp suricata.yaml suricata.yaml.backup
sudo nano suricata.yaml
```

## Sample YAML configuration:

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

## Creating Custom Rules

```sh
cd /var/lib
sudo mkdir -p suricata/rules
sudo touch /var/lib/suricata/rules/custom.rules
```

## Example Rule

```sh
sudo nano /var/lib/suricata/rules/custom.rules
```

```sh
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

## Lab: Detect Nmap Scan

```sh
tail -f /var/log/suricata/fast.log
nmap -sS <TARGET_IP>
```

## Edit `suricata.yaml` to enable NFQUEUE

```sh
nfq:
  mode: accept
  repeat-mark: 1
  repeat-mask: 1
  route-queue: 2

af-packet:
  - interface: enp0s8
```

## Modify Rules to Use drop

```sh
sudo nano /var/lib/suricata/rules/custom.rules
```

```sh
drop icmp any any -> $HOME_NET any (msg: "Drop ICMP packets!"; sid:124; rev:1;)
```

## Configure IPTables to Use NFQUEUE`

```sh
sudo iptables -I INPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -I OUTPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -vnL
```

## Start Suricata in IPS Mode

```sh
sudo suricata -c /etc/suricata/suricata.yaml -q 2
```

## Test the Rule

```sh
ping <Suricata_IP>
tail -f /var/log/suricata/fast.log
```

## Remove IPTables Rules (Optional Cleanup)

```sh
sudo iptables -D INPUT -p icmp -j NFQUEUE --queue-num 2
sudo iptables -D OUTPUT -p icmp -j NFQUEUE --queue-num 2
```
