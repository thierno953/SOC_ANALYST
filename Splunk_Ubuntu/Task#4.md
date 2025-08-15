# Detection of Abnormal Network Traffic

- Install and configure Suricata IDS

- Integrate logs into Splunk

- Simulate a malicious scan

- Visualize and analyze alerts

- Respond to the incident

#### Installing and configuring Suricata

- [Proofpoint Emerging Threats Rules](https://rules.emergingthreats.net/)

```sh
apt-get update
add-apt-repository ppa:oisf/suricata-stable -y
apt-get install suricata -y
```

#### Download Emerging Threats (ET) rules:

```sh
cd /etc/suricata && mkdir rules && cd
cd /tmp/
curl -LO https://rules.emergingthreats.net/open/suricata-7.0.3/emerging.rules.tar.gz
tar -xvzf emerging.rules.tar.gz
mv rules/*.rules /etc/suricata/rules/
chmod 640 /etc/suricata/rules/*.rules
```

#### Suricata configuration file (`/etc/suricata/suricata.yaml`)

```sh
vars:
  address-groups:
    HOME_NET: "[IP_ADDRESS]"
    EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"

af-packet:
  - interface: enp0s3
```

- **Note**: Replace [`IP_ADDRESS`] with the actual IP address of the target machine (`victim`).

#### Starting and testing Suricata

```sh
systemctl enable --now suricata
systemctl restart suricata
systemctl status suricata
tail -f /var/log/suricata/fast.log
tail -f /var/log/suricata/eve.json | jq .
grep -i "SCAN" /etc/suricata/rules/*.rules
```

## Configure Splunk Universal Forwarder

- Edit `inputs.conf`

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Content of `inputs.conf`

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
index = linux_file_integrity
sourcetype = auditd

[monitor:///var/log/suricata/eve.json]
disabled = false
index = network_security_logs
sourcetype = suricata
```

#### Restart the Forwarder

```sh
/opt/splunkforwarder/bin/splunk restart
```

## Simulate an attack (TCP SYN scan)

#### From the attacker machine:

```sh
nmap -sS -p 1-1000 --reason <victim_IP>
```

## Analyze Alerts in Splunk

- Search for alerts by attacker's IP:

```sh
index="network_security_logs" sourcetype="suricata" src_ip="<attacker_IP>"
```

## Responding to the Incident

- Block attacker IP at the firewall:

```sh
iptables -A INPUT -s <attacker_IP> -j DROP
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_14.png)
