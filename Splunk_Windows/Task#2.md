# Investigation of PowerShell Abuse on Windows Machines

- Verify PowerShell and Sysmon log collection
  - Viewing PowerShell logs:
  - Open Windows Event Viewer:

`Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

![Enterprise](/Splunk_Windows/assets/splunk_windows_05.png)

#### Splunk Universal Forwarder Configuration

- File `C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

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

#### Restart Splunk service

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

#### PowerShell attack simulation

> [Download the EICAR test file (standard anti-malware test) to simulate suspicious activity](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
PS C:\Users\Administrator> schtasks /create /tn "MaliciousTasks" /tr "C:\Malware.exe" /sc once /st 12:00
PS C:\Users\Administrator> schtasks /run /tn "MaliciousTasks"
```

#### Investigation queries in Splunk

- General PowerShell activity search

```sh
index="powershell_logs" sourcetype="WinEventLog:PowerShell"
```

- Specific search related to the `EICAR` file

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*eicar*"
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_06.png)

## Incident response

#### Blocking outbound via Windows Defender Firewall

- GUI:

  - `Start > Windows Defender Firewall > Outbound Rules > New Rule`

  - Type: Program

  - Target: All programs or PowerShell only

  - Action: Block the connection

  - Profiles: Domain, Private, Public

  - Name the rule, e.g. "Block PowerShell Internet Access"

#### Blocking via PowerShell

```sh
# Block all outbound traffic (use with caution)
New-NetFirewallRule -DisplayName "Block All Traffic" -Direction Outbound -Action Block
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_07.png)
