# Task 1: Investigating RDP Brute-Force Attacks on Windows Login

- Installing Sysmon
- Setting up Splunk
- Simulate Attacks & Visualize
- Incident Response

#### Installing Sysmon

- [Sysmon for Windows](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [sysmon-config | A Sysmon configuration file](https://github.com/SwiftOnSecurity/sysmon-config)

```sh
# Installer Sysmon avec la configuration SwiftOnSecurity
cd C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

cd C:\Users\Administrator\Downloads\Sysmon> Get-Service Sysmon*

# Modifier la configuration si nécessaire
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml

PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

`Start > Event Viewer > Application and Service Logs > Microsoft > Windows > Sysmon > Operational`

#### Setting up Splunk

`C:\Program Files > SplunkUniversalForwarder > etc > system > local > outputs.conf`

```sh
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <IP_de_l'indexeur>:9997

[tcpout-server://<IP_de_l'indexeur>:9997]
```

`C:\Program Files > SplunkUniversalForwarder > etc > system > local > inputs.conf`

```sh
[WinEventLog://Application]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog

[WinEventLog://Security]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog
source = WinEventLog:Security

[WinEventLog://System]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog
source = WinEventLog:System

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog
source = XmlWinEventLog:Sysmon
renderXml = false
```

`C:\Program Files > SplunkUniversalForwarder > etc > system > local > props.conf`

```sh
[XmlWinEventLog:Sysmon]
KV_MODE = xml
TRANSFORMS-sysmon = sysmon-eventid,sysmon-data
```

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe status
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe list forward-server
```

```sh
index=sysmon_logs sourcetype=XmlWinEventLog
index=sysmon_logs sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon"
index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:Application"
index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:Security"
index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:System"

index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:Application" | stats count by EventCode
index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:Security" | stats count by EventCode
index=windows_event_logs sourcetype=WinEventLog source="WinEventLog:System" | stats count by EventCode
index=sysmon_logs sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon" | stats count by EventCode


index=windows_event_logs | stats count by sourcetype, source
index=sysmon_logs | stats count by sourcetype, source
index=sysmon_logs sourcetype=XmlWinEventLog | stats count by EventCode

index=sysmon_logs sourcetype=XmlWinEventLog source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
index=sysmon_logs | stats count by sourcetype
index=sysmon_logs sourcetype=XmlWinEventLog EventCode=3 | table _time, host, user, dest_ip, dest_port
index=sysmon_logs sourcetype=XmlWinEventLog EventCode=1 | table _time, host, user, parent_process_name, process_name
index=sysmon_logs sourcetype=XmlWinEventLog EventCode=1 | table _time, host, process_name, parent_process_name
```

#### Simulate the attack and Visualize on Splunk Dashboard

```sh
root@attack:~# apt install hydra -y
root@attack:~# hydra -l administrator -P password.txt <IP_VICTIME> rdp
```

####Search & Reporting

```sh
index=sysmon_logs sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon"
index=sysmon_logs sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon" SourceIp="<IP_VICTIME>"
```

#### Incident Response

- To block inbound connections on port 3389 (RDP) using Windows Defender Firewall

`Start > Windows Defender Firewall > Inbound Rules > New Rules > Ports > TCP (Specific local ports: 3389) > Block the connection > (Domain, Private, and Public) > Provide a Name for the "Block RDP Brute Force" `

```sh
PS C:\Users\Administrator> New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP_ATTACK>
```
