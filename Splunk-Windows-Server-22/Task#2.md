# Investigating PowerShell Abuse on Windows Machines

- Objective
- Sysmon(If not installed)
- Set up Splunk(if needed)
- Simulate the attack and visualize the event
- Incident Response

# 1 - Verify Sysmon Installation

![splunk](/Splunk-Windows-Server-22/assets/22.png)

# 2 - Setting up Splunk

![splunk](/Splunk-Windows-Server-22/assets/23.png)

inputs.conf

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

[WinEventLog://Microsoft-Windows-PowerShell/Operational]
disabled = 0
index = powershell_logs
sourcetype = WinEventLog:PowerShell
renderXml = false
```

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

(Settings --> System --> Server controls --> restart splunk)

###### Search & Reporting

```sh
index="powershell_logs" sourcetype="WinEventLog:PowerShell"
```

# 3 - Simulate the attack and Visualize on Splunk Dashboard

# https://www.eicar.org/download-anti-malware-testfile/

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

###### Search & Reporting

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*eicar*"
```

# 4 - Incident Response

```sh
PS C:\Users\Administrator> New-NetFirewallRule -DisplayName "Block All Traffic" -Direction Outbound -Action Block
```
