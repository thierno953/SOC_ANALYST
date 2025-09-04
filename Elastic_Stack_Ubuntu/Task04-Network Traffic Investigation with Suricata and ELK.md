# Network Traffic Investigation with Suricata and ELK

#### Objective

- Detect malicious or suspicious network activities using **Suricata** (IDS/IPS) and visualize the events in the **ELK Stack** (Elasticsearch, Logstash, Kibana).

## Installing and Configuring Suricata

- **Suricata** is an open-source Intrusion Detection System (IDS) and Intrusion Prevention System (IPS). It analyzes network traffic in real-time and can identify **malicious behavior, attacks, or anomalies** using detection rules (such as **Emerging Threats** rules).

- In this project, Suricata is used to **monitor network communications on the target host** and forward security alerts to **ELK SIEM**. This enables analysts to **detect port scans, suspicious connections, or unusual file transfers**, and respond quickly.

- [Emerging Threats Rules Link](https://rules.emergingthreats.net/)

```sh
sudo apt-get update
sudo add-apt-repository ppa:oisf/suricata-stable -y
sudo apt-get install suricata -y
```

- Add detection rules:

```sh
cd /etc/suricata && mkdir rules
cd /tmp
curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz
sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

- Configuration of `suricata.yaml`

```sh
vars:
  address-groups:
    HOME_NET: "[ELASTIC_AGENT_HOST_IP]"   # IP address of the monitored host
    EXTERNAL_NET: "any"

af-packet:
  - interface: enp0s3

default-rule-path: /etc/suricata/rules
rule-files:
  - "*.rules"

stats:
  enabled: yes
```

- Start the service:

```sh
sudo systemctl restart suricata
sudo systemctl status suricata
```

- View logs:

```sh
tail -f /var/log/suricata/fast.log
tail -f /var/log/suricata/eve.json

nano /etc/suricata/rules/emerging-malware.rules
```

## Integrate Suricata with ELK

- In Kibana ->` Management > Integrations`

- Search for **Suricata** -> Add the integration

- File path: `/var/log/suricata/eve.json`

- Select existing agent -> **Save and Continue**

#### Dashboards :

- `[Logs Suricata] Alert Overview`

- `[Logs Suricata] Events Overview`

## Simulate an Attack and Visualize Alerts

- From another machine (attacker):

```sh
nmap -sS <ELASTIC_AGENT_HOST_IP>
nmap -A -T4 <ELASTIC_AGENT_HOST_IP>
```

- On the monitored host:

```sh
tail -f /var/log/suricata/eve.json
tail -f /var/log/suricata/fast.log
```

#### Analysis in Kibana Go to `Discover > KQL Queries`:

```sh
suricata.eve.alert.signature_id: *
```

![ELK](/Elastic_Stack_Ubuntu/assets/18.png)

```sh
source.address: "<ATTACKER_IP>"
```

![ELK](/Elastic_Stack_Ubuntu/assets/19.png)

#### Incident Response

- Identify the signature and source IP address

- Check whether the activity is legitimate or malicious

- Isolate the source host or block the traffic if necessary

- Update Suricata rules if required
