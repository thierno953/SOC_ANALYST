# Task #4: Detecting Abnormal Network Traffic

- Installing Suricata IDS
- Setting up Splunk
- Similate Attack & Visualize
- Incident Response

#### Installing and setting up Suricata

- [Proofpoint Emerging Threats Rules](https://rules.emergingthreats.net/)

```sh
root@node01:/# apt-get update
root@node01:/# add-apt-repository ppa:oisf/suricata-stable -y
root@node01:/# apt-get install suricata -y
root@node01:/# cd /etc/suricata && mkdir rules && cd
root@node01:~# cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-7.0.3/emerging.rules.tar.gz

root@node01:/tmp# sudo tar -xvzf emerging.rules.tar.gz && sudo mv rules/*.rules /etc/suricata/rules/
root@node01:/tmp# sudo chmod 640 /etc/suricata/rules/*.rules
root@node01:/tmp# cd /etc/suricata/rules/
root@node01:/etc/suricata/rules# nano emerging-malware.rules
```

```sh
root@node01:/etc/suricata/rules# nano /etc/suricata/suricata.yaml
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
root@node01:/etc/suricata/rules# systemctl restart suricata
root@node01:/etc/suricata/rules# systemctl status suricata
root@node01:/etc/suricata/rules# suricata-update
root@node01:/etc/suricata/rules# cd /var/log/suricata/
root@node01:/var/log/suricata# tail -f fast.log
```

#### Setting up Splunk

```sh
root@node01:/var/log/suricata# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

[monitor:///var/log/audit/audit.log]
disabled = false
sourcetype = auditd
index = linux_file_integrity

[monitor:///var/log/suricata/fast.log]
disabled = false
index = network_security_logs
sourcetype = suricata
```

```sh
root@node01:/var/log/suricata# /opt/splunkforwarder/bin/splunk restart
```

#### Simulating an attack and Analyzing logs on Splunk

```sh
root@attack:~# nmap -sS <IP_VICTIM_MACHINE>
```

####Search & Reporting

```sh
index="network_security_logs" sourcetype="suricata"
index="network_security_logs" sourcetype="suricata" src_ip="<IP_ATTACK_MACHINE>"
```
