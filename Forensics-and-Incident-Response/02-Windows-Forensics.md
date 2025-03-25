# Windows Forensics

- The practice of collecting and analyzing data from Windows systems to identify, investigate, and recover from security incidents.

- **Purpose:**

  - Identify malicious activities
  - Recover evidence
  - Prevent further attacks

- Focus Areas: Filesystem, registry, event logs, memory, network activity.

## Incident Response Process Overview

#### Key Components of Windows Forensics

- Filesystem: Investigating files, folders, timestamps, and hidden data.
- Registry: Tracting system configuration changes, user activity, and application settings.
- Event Logs: Examining security, application,and system logs for relevant events.
- Memory (RAM): Capturing volatile data, including running processes and network connections.

#### Why Windows Forensics is Important

- Incident Response: Detect and analyze the scope and impact of breaches.
- Legal Evidence: Ensure data integrity for court-admissible digital evidence.
- Recovery: Help restore systems to ensure, operational status after an attack.
- Proactive Security: Improve defenses by learning from past incidents.

#### Why Use PowerSell for Forensics

- Powerful Command-Line Tool: Built into Windows, making it accessible for forensic investigations.
- Automation Capabilities: Easily automate repetitive forensic tasks and incident response procedures.
- System-wide Access: Direct access to files, processes, logs, and network activity without installing third-party tools.
- Scripting Flexibility: Create custom scripts for specific forensic needs.

#### Key PowerSell CMDLETS for Forensics

- **Get-Process**: List all running processes and their details (memory usage, CPU time, etc.).
- **Get-EventLog**: Retrieve and filter windows event logs (Security, Application, System logs).
- **Get-FileHash**: Generate cryptographic hashes (MD5,SHA256) to verify file integrity.
- **Get-ChildItem**: Recursively list files and folders with metadata for forensic analysis.
- **Get-WmiObject**: Access system information (e.g, installed software, startup items, system configurations).

#### 1 - Collecting System Information with PowerShell

```sh
PS C:\Users\Administrator> Get-ComputerInfo
PS C:\Users\Administrator> Get-ComputerInfo | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\SystemInfo.txt
PS C:\Users\Administrator> Get-Process
PS C:\Users\Administrator> Get-Process | Format-Table -AutoSize
PS C:\Users\Administrator> Get-Process | Format-Table -AutoSize | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessList.txt
```

#### 2 - Investigating User Accounts

- List all AD User ACcounts

```sh
PS C:\Users\Administrator> Get-ADUser -Filter *
PS C:\Users\Administrator> Get-ADUser -Filter * | Select-Object Name, Enabled | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ADUserAccounts.txt
```

- List members of the Domain Admin group

```sh
PS C:\Users\Administrator> Get-ADGroupMember -Identity "Domain Admins"
```

- Get detailed into for a specific AD User

```sh
PS C:\Users\Administrator> Get-ADUser -Identity "thierno" -Properties *
```

#### 3 - Investigating Process

```sh
PS C:\Users\Administrator> Get-Process

PS C:\Users\Administrator> Get-Process | Where-Object { $_.Name -notin @('explorer', 'svchost', 'winlogon', 'lsass', 'services')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousProcesses.txt"

PS C:\Users\Administrator> Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath

PS C:\Users\Administrator> Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindows.txt

PS C:\Users\Administrator> Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindowsBrief.txt

PS C:\Users\Administrator> Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU

PS C:\Users\Administrator> Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\HighCPUProcesses.txt

PS C:\Users\Administrator> Get-Process | Sort-Object PM -Descending | Select-Object -First 10 Name, Id, PM
```

#### 4 - Investigating Services

- List all Services
- Detaild info of a specific service
- Services running from non-standard paths

```sh
PS C:\Users\Administrator> Get-Service

PS C:\Users\Administrator> Get-Service | Format-Table -AutoSize

PS C:\Users\Administrator> Get-Service | Where-Object { $_.Name -notin @('TrustedInstaller', 'WindDefend', 'EventLog', 'Dhcp', 'Dnscache')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousServices.txt"

PS C:\Users\Administrator> Get-Service -Name "BITS" | Format-List *

PS C:\Users\Administrator> Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName

PS C:\Users\Administrator> Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ServiceOutOfWindows.txt
```

#### 5 - Checking Scheduled Tasks

```sh
PS C:\Users\Administrator> Get-ScheduledTask | Format-Table -AutoSize

PS C:\Users\Administrator> Get-ScheduledTask | Where-Object { $_.TaskName -notin @('UpdateTask', 'SystemTasks', 'WindowsTasks')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousScheduledTasks.txt"
```

#### 6 - Checking Active Internet Connections

```sh
PS C:\Users\Administrator> Get-NetTCPConnection

PS C:\Users\Administrator> Get-NetTCPConnection | Format-Table -AutoSize

PS C:\Users\Administrator> Get-NetTCPConnection | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ActiveNetTCPConnections.txt"

PS C:\Users\Administrator> Get-NetTCPConnection -LocalPort 3389

PS C:\Users\Administrator> Get-NetTCPConnection -LocalPort 3389 |  Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\RDPConnection.txt"

PS C:\Users\Administrator> Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}

PS C:\Users\Administrator> Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize

PS C:\Users\Administrator> Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses.txt"

PS C:\Users\Administrator> Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses1.txt"
```
