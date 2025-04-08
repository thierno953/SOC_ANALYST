# Task 2: Investigating PowerShell Abuse on Windows Machines

- Simulate the attack and Visualize the event
- Incident Response

#### Verify Sysmon Installation

`Start > Event Viewer > Application and Service Logs > Microsoft > Windows > PowerShell > Operational`

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

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = powershell_logs
source = WinEventLog
sourcetype = WinEventLog:PowerShell
renderXml = false
```

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

```sh
index="powershell_logs" sourcetype="WinEventLog" source="WinEventLog:Microsoft-Windows-PowerShell/Operational"
```

#### Simulate the attack and Visualize on Splunk Dashboard

- [Anti Malware Testfile](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

####Search & Reporting

```sh
index=sysmon_logs sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon" "*eicar*"
```

#### Incident Response

`Start > Windows Defender Firewall > Outbound Rules > New Rules > Program > All programs > Block the connection > (Domain, Private, and Public) > Provide a Name for the "Block All Traffi"`

```sh
PS C:\Users\Administrator> New-NetFirewallRule -DisplayName "Block All Traffic" -Direction Outbound -Action Block
```
