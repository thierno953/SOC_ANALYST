# Task#4: Network Traffic Investigation with Suricata and ELK

- Install suricata
- Prepare ELK for detection
- Similate the attack and visualize the events
- Incident Response

#### Installing and setting up Suricata

- [Proofpoint Emerging Threats Rules](https://rules.emergingthreats.net/)

```sh
root@fleet-agent:~# sudo apt-get update
root@fleet-agent:~# sudo add-apt-repository ppa:oisf/suricata-stable -y
root@fleet-agent:~# sudo apt-get install suricata -y
root@fleet-agent:~# cd /etc/suricata && mkdir rules && cd
root@fleet-agent:~# cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
root@fleet-agent:/tmp# sudo tar -xvzf emerging.rules.tar.gz && sudo mv rules/*.rules /etc/suricata/rules/
root@fleet-agent:/tmp# sudo chmod 640 /etc/suricata/rules/*.rules
cd /etc/suricata/rules/
root@fleet-agent:/etc/suricata/rules# nano emerging-malware.rules
```

```sh
root@fleet-agent:/etc/suricata/rules# sudo nano /etc/suricata/suricata.yaml
```

```sh
rule-files:
  - emerging-web_client.rules
  - emerging-malware.rules
  - emerging-attack_response.rules
  - emerging-trojan.rules

# ===============================

vars:
  # more specific is better for alert accuracy and performance
  address-groups:
    HOME_NET: "[IP_ADDRESS]"

    #EXTERNAL_NET: "!$HOME_NET"
    EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules

rule-files:
  - "*.rules"

stats:
  enabled: yes

af-packet:
  - interface: enp0s3
```

```sh
root@fleet-agent:~# sudo systemctl restart suricata
root@fleet-agent:~# sudo systemctl status suricata
root@fleet-agent:~# suricata-update
root@fleet-agent:~# tcpdump -i enp0s3
root@fleet-agent:~# cd /var/log/suricata/
root@fleet-Agent:/var/log/suricata# tail -f fast.log
root@fleet-agent:~# cd /etc/suricata/rules
root@fleet-agent:/etc/suricata/rules~# nano emerging-malware.rules
```

#### Prepare ELK for detection

`Management > Integrations > Add suricata`

```sh
(Paths: /var/log/suricata/eve.json > existing hosts > save and continue)
(Suricata --> assets --> Dashboards [Logs Suricata] Alert Overview)
(Suricata --> assets --> Dashboards [Logs Suricata] Events Overview)
```

#### Simulate the attack and visualize the events

```sh
root@attack:~# nmap -sS <IP_FLEET_AGENT>
```

```sh
root@fleet-agent:~# cd /var/log/suricata/
root@fleet-Agent:/var/log/suricata# tail -f eve.json
root@fleet-Agent:/var/log/suricata# tail -f fast.log
```

- `Analystics > Discover`

```sh
suricata.eve.alert.signature_id
source.address: "<IP_ADDRESS>"
```
