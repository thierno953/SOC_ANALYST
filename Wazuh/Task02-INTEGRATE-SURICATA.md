# INTEGRATE SURICATA WITH WAZUH

## Install Suricata and Download Rules

- Reference: [https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html](https://documentation.wazuh.com/current/proof-of-concept-guide/integrate-network-ids-suricata.html)

```sh
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y

cd /tmp/ 
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mkdir -p /etc/suricata/rules
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
sudo chown root:suricata /etc/suricata/rules/*.rules
```

## Add Custom Detection Rules

- Edit `/etc/suricata/rules/local.rules`:

```sh
alert tcp any any -> any any (msg:"[CUSTOM] Nmap SYN Scan Detected"; flags:S; threshold:type both, track by_src, count 10, seconds 30; sid:10000001; rev:1;)
alert icmp any any -> any any (msg:"[CUSTOM] ICMP Flood Detected"; itype:8; threshold:type both, track by_src, count 100, seconds 10; sid:10000002; rev:1;)
alert tcp any any -> any any (msg:"[CUSTOM] TCP SYN Flood Detected"; flags:S; threshold:type both, track by_src, count 50, seconds 10; sid:10000003; rev:1;)
```

## Configure Suricata

- Edit `/etc/suricata/suricata.yaml`:

```sh
HOME_NET: "<UBUNTU_IP>"
EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"
  - local.rules

stats:
  enabled: yes

af-packet:
  - interface: enp0s3
```

## Restart Suricata

```sh
sudo systemctl restart suricata
```

## Integrate Suricata with Wazuh

- Edit `/var/ossec/etc/ossec.conf`:

```sh
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

```sh
sudo systemctl restart wazuh-agent
```

## Monitoring & Testing

- Monitor Suricata logs:

```sh
tail -f /var/log/suricata/fast.log
tail -f /var/log/suricata/eve.json | jq .
```

## Launch Attacks for Testing Detection

- On attacker machine:

```sh
nmap -sS -T5 <Wazuh agent>
ping -c 100 <Wazuh agent>
sudo hping3 -1 --flood <Wazuh agent>
sudo hping3 --icmp --flood <Wazuh agent>
sudo hping3 -S --flood --rand-source -p 80 <Wazuh agent>
```

## Visualize the alerts

```sh
rule.groups:suricata
```

![WAZUH](/Wazuh/assets/02.png)

![WAZUH](/Wazuh/assets/03.png)
