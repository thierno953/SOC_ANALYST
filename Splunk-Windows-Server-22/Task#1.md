# Investigating RDP Brute-Force Attacks on Windows Login

- Objective
- Installing Sysmon
- Setting up Splunk
- Simulate attack & visuailize
- Incident Response

#### 1 - Installing Sysmon

```sh
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
https://github.com/SwiftOnSecurity/sysmon-config
https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml
```

![splunk](/Splunk-Windows-Server-22/assets/01.png)
![splunk](/Splunk-Windows-Server-22/assets/02.png)

```sh
# Installer Sysmon avec la configuration SwiftOnSecurity
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -i .\sysmonconfig-export.xml

# Modifier la configuration si nécessaire
PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -c .\sysmonconfig-export.xml

PS C:\Users\Administrator\Downloads\Sysmon> .\Sysmon64.exe -u .\sysmonconfig-export.xml
```

![splunk](/Splunk-Windows-Server-22/assets/03.png)
![splunk](/Splunk-Windows-Server-22/assets/04.png)

#### 2 - Setting up Splunk

(Program Files --> SplunkUniversalForwarder --> etc --> System --> local --> inputs.conf)

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

Applications et services Logs > Microsoft > Windows > Sysmon > Operational

```sh
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe status
```

(Settings --> System --> Server controls --> restart splunk)

###### Search & Reporting

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon"
```

#### 3 - Simulate the attack and Visualize on Splunk Dashboard

```sh
# Installer Hydra
apt install hydra -y

# Lancer l'attaque
hydra -l administrator -P password.txt <IP WINDOWS SPLUNK SERVER> rdp
```

###### Search & Reporting

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon"
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" sourceIp="<IP Attach Machine>"
```

# 4 - Incident Response

![splunk](/Splunk-Windows-Server-22/assets/05.png)
![splunk](/Splunk-Windows-Server-22/assets/06.png)
![splunk](/Splunk-Windows-Server-22/assets/07.png)
![splunk](/Splunk-Windows-Server-22/assets/08.png)
![splunk](/Splunk-Windows-Server-22/assets/09.png)
![splunk](/Splunk-Windows-Server-22/assets/10.png)
![splunk](/Splunk-Windows-Server-22/assets/11.png)
![splunk](/Splunk-Windows-Server-22/assets/12.png)
![splunk](/Splunk-Windows-Server-22/assets/13.png)
![splunk](/Splunk-Windows-Server-22/assets/14.png)
![splunk](/Splunk-Windows-Server-22/assets/15.png)
![splunk](/Splunk-Windows-Server-22/assets/16.png)
![splunk](/Splunk-Windows-Server-22/assets/17.png)
![splunk](/Splunk-Windows-Server-22/assets/18.png)
![splunk](/Splunk-Windows-Server-22/assets/19.png)
![splunk](/Splunk-Windows-Server-22/assets/20.png)

- Blocage de l'IP attaquante

```sh
PS C:\Users\Administrator> New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP ATTACK>
```

![splunk](/Splunk-Windows-Server-22/assets/21.png)

