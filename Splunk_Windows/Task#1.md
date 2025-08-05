# Investigating RDP Brute Force Attacks on Windows Connections

#### Installation and Configuration of Sysmon

- [Sysmon for Windows](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

- [Recommended Sysmon Configuration](https://github.com/SwiftOnSecurity/sysmon-config)

#### Installation commands

```sh
# Install Sysmon with recommended configuration
C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

# Check Sysmon service status
C:\Users\Administrator\Downloads\Sysmon> Get-Service Sysmon*

# Update Sysmon configuration
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml

# Uninstall or update configuration
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

- Viewing logs:
  - Open Windows `Event Viewer`:

`Start > Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

![Enterprise](/Splunk_Windows/assets/splunk_windows_02.png)

## Configure Splunk Universal Forwarder

- Edit `inputs.conf` to collect Sysmon and other logs:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

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

#### Managing Splunk Universal Forwarder service:

```sh
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe restart
.\splunk.exe status
.\splunk.exe list forward-server
```

## Simulate RDP brute force attack

- On a Linux attacker machine:

```sh
apt install hydra -y
hydra -l administrator -P password.txt <IP_VICTIM_MACHINE> rdp
```

#### Search query in Splunk (Search & Reporting)

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" SourceIp="<ATTACKER_IP>"
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_03.png)

## Incident response

#### Block via Windows Defender Firewall GUI

- Open: `Start > Windows Defender Firewall > Inbound Rules`

- Create a new rule:

  - Type: TCP Ports

  - Specific local port: 3389

  - Action: Block the connection

  - Profiles: Domain, Private, Public

  - Name the rule: "Block RDP Brute Force"

#### Block via PowerShell

```sh
New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <ATTACKER_IP>
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_04.png)
