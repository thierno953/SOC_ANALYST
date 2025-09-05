## Objective

The objective of this task is to detect, investigate, and respond to brute-force attacks targeting the Remote Desktop Protocol (RDP) service on a Windows machine. This task uses Sysmon and Splunk to monitor failed and successful login attempts and identify suspicious patterns indicative of an attack.

## Steps

1. Setting up Windows Machine
2. Setting up Splunk
3. Simulate the attack and Visualize the alert
4. Incident Response

### Installing Sysmon

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

### 1. Setting up Windows Machine

- Edit the inputs.conf file

```
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = sysmon_logs
sourcetype = XmlWinEventLog:Sysmon
renderXml = false

```

### 2. Setting up Splunk

- Create a Sourcetype(if required)
- Install Sysmon App from Splunkbase

```
Add-LocalGroupMember -Group "Event Log Readers" -Member "NT SERVICE\SplunkForwarder"
net stop splunkforwarder
net start splunkforwarder
```

### 3. Simulate the attack and Visualize the alert

- Use manual login attempts or tools liek Hydra

```
hydra -l administrator -P /path/to/passwords.txt rdp://<windows-machine-ip>
```

![Splunk](/Splunk_Windows/assets/04.png)

![Splunk](/Splunk_Windows/assets/05.png)

- Visualize the event on Splunk Dashboard

```
index="sysmon_logs"  sourcetype="XmlWinEventLog:Sysmon" SourceIp="<IP>"
```

![Splunk](/Splunk_Windows/assets/06.png)

### 4. Incident Response

### **Incident Response**

1. **Block Malicious IPs**:

- Use the Windows Firewall to block suspicious IP addresses:

```
New-NetFirewallRule -DisplayName "Block RDP Brute Force" -Direction Inbound -Action Block -RemoteAddress <IP>

```

![Splunk](/Splunk_Windows/assets/07.png)

2. **Reset Compromised Accounts**:
   - Reset passwords for compromised accounts:

```
       net user <username> <new_password>

```

3. **Enable Account Lockout Policy**:
   - Set a lockout policy to prevent repeated brute-force attempts:

```
        secpol.msc
```

        - Navigate to `Account Lockout Policy` and configure:
            - Lockout Threshold: 5 failed attempts.
            - Lockout Duration: 15 minutes.
