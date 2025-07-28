# Windows Forensics

- The practice of collecting and analyzing data from Windows systems to identify, investigate, and recover from security incidents.

  - Identify malicious activities

  - Recover evidence

  - Prevent further attacks

- **Focus Areas**: `filesystem, registry, event logs, memory, network activity`

## Key Components of Windows Forensics

- **Filesystem**: Investigating files, folders, timestamps, and hidden data.

- **Registry**: Tracking system configuration changes, user activity, and application settings.

- **Event Logs**: Examining security, application, and System logs for relevant events.

- **Memory (RAM)**: Capturing volatile data, including running processes and network connections.

## Why Windows Forensics is Important

- **Incident Response**: Detect and analyze the scope and impact of a breaches.

- **Legal Evidence**: Ensure data integrity for court-admissible digital evidence.

- **Recovery**: Help restore systems to secure, operational status after on attack.

- **Proactive Security**: Improve defenses by learning from past incidents.

## Why Use PowerShell for Forensics

- **Powerful Command-Line Tool**: Built into Windows, making it accessible for forensic investigations.

- **Automation Capabilities**: Easily automate repetitive forensic tasks and incident response procedures.

- **System-wide Access**: Direct access to files, processes, logs, and network activity without installing third-party tools.

- **Scripting Flexibility**: Create custom scripts for specific forensic needs.

## Key PowerShell Cmdlets for Forensics

- `Get-Process`: List all running processes with their details (`memory usage, CPU time, etc`).

- `Get-EventLog`: Retrieve and filter Windows event logs (`Security, Application, System logs`).

- `Get-FileHash`: Generate cryptographic hashes (`MD5, SHA256`) to verify file integrity.

- `Get-ChildItem`: Recursively list files and folders with metadata for forensic analysis.

- `Get-WmiObject`: Access system information (installed software, startup items, system configurations).

## Collecting System Information with PowerShell

```sh
Get-ComputerInfo
Get-ComputerInfo | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\SystemInfo.txt
Get-Process
Get-Process | Format-Table -AutoSize
Get-Process | Format-Table -AutoSize | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessList.txt
```

## Investigating User Accounts

#### List all AD user accounts:

```sh
Get-ADUser -Filter *
Get-ADUser -Filter * | Select-Object Name, Enabled | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ADUserAccounts.txt
```

#### List `Domain Admins` group members:

```sh
Get-ADGroupMember -Identity "Domain Admins"
```

#### Get detailed info on a specific user:

```sh
Get-ADUser -Identity "thierno" -Properties *
```

## Investigating Process

```sh
Get-Process

Get-Process | Where-Object { $_.Name -notin @('explorer', 'svchost', 'winlogon', 'lsass', 'services')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousProcesses.txt"

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindows.txt

Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Select-Object Name, ProcessId, ExecutablePath | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ProcessOutOfWindowsBrief.txt

Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU

Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, Id, CPU | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\HighCPUProcesses.txt

Get-Process | Sort-Object PM -Descending | Select-Object -First 10 Name, Id, PM
```

## Investigating Services

```sh
Get-Service

Get-Service | Format-Table -AutoSize

Get-Service | Where-Object { $_.Name -notin @('TrustedInstaller', 'WindDefend', 'EventLog', 'Dhcp', 'Dnscache')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousServices.txt"

Get-Service -Name "BITS" | Format-List *

Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName

Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Select-Object Name, DisplayName, PathName | Out-File -FilePath C:\Users\Administrator\Documents\221B-Case\ServiceOutOfWindows.txt
```

## Checking Scheduled Tasks

```sh
Get-ScheduledTask | Format-Table -AutoSize

Get-ScheduledTask | Where-Object { $_.TaskName -notin @('UpdateTask', 'SystemTasks', 'WindowsTasks')} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\SuspicousScheduledTasks.txt"
```

## Checking Active Internet Connections

```sh
Get-NetTCPConnection

Get-NetTCPConnection | Format-Table -AutoSize

Get-NetTCPConnection | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ActiveNetTCPConnections.txt"

Get-NetTCPConnection -LocalPort 3389

Get-NetTCPConnection -LocalPort 3389 | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\RDPConnection.txt"

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses.txt"

Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize | Out-File -FilePath "C:\Users\Administrator\Documents\221B-Case\ConnectionwithProcesses1.txt"
```
