# Task 1: Investigating RDP Brute-Force Attacks on Windows Login

> Installing Sysmon
> Setting up Splunk
> Simulate Attacks & Visualize
> Incident Response

#### Installing Sysmon

> [Sysmon for Windows](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

> [sysmon-config | A Sysmon configuration file](https://github.com/SwiftOnSecurity/sysmon-config)

```sh
# Installer Sysmon avec la configuration SwiftOnSecurity
C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

C:\Users\Administrator\Downloads\Sysmon> Get-Service Sysmon*

# Modifier la configuration si nécessaire
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml

PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

> `Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

#### Setting up Splunk

> `C:\Program Files > SplunkUniversalForwarder > etc > system > local > outputs.conf`

```sh
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <IP_DE_L_INDEXEUR>:9997

[tcpout-server://<IP_DE_L_INDEXEUR>:9997]
```

> `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```sh
[WinEventLog://Application]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Application

[WinEventLog://Security]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Security

[WinEventLog://System]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:System

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog:Sysmon
renderXml = false
```

```sh
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe restart
.\splunk.exe status
.\splunk.exe list forward-server
```

#### Simulate the attack and Visualize on Splunk Dashboard

```sh
# Sur la machine d'attaque (Linux)
apt install hydra -y
hydra -l administrator -P password.txt <IP_VICTIME> rdp
```

> `Search & Reporting`

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" sourceIp="<IP_Attach_Machine>"
```

#### Incident Response

> To block inbound connections on port 3389 (RDP) using Windows Defender Firewall

`Start > Windows Defender Firewall > Inbound Rules > New Rules > Ports > TCP (Specific local ports: 3389) > Block the connection > (Domain, Private, and Public) > Provide a Name for the "Block RDP Brute Force"`

> PowerShell

`New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP_ATTACK>`
