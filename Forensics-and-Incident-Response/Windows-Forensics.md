# Windows Forensics

- Windows Forensics is the practice of collecting and analyzing data from Windows systems to identify, investigate, and recover from security incidents. The main objectives are:

  - Identify malicious activities

  - Recover evidence

  - Prevent further attacks

- **Focus areas**: `filesystem, registry, event logs, memory, network activity`

## Key Components of Windows Forensics

- **Filesystem**: Investigate files, folders, timestamps, and hidden data to detect malicious activity.

- **Registry**: Track system configuration changes, user activity, and application settings.

- **Event Logs**: Examine security, application, and system logs for authentication, malware, or unauthorized access.

- **Memory (RAM)**: Capture volatile data such as running processes and network connections.

- **Network Activity**: Monitor active connections, open ports, and suspicious network traffic.

## Importance of Windows Forensics

- **Incident Response**: Detect and analyze the scope and impact of breaches.

- **Legal Evidence**: Preserve integrity for court-admissible digital evidence.

- **Recovery**: Restore systems to secure, operational status after an attack.

- **Proactive Security**: Improve defenses by learning from past incidents.

## Why Use PowerShell for Forensics

- Built-in command-line tool in Windows.

- Automation of repetitive tasks.

- System-wide access without third-party tools.

- Flexible scripting for custom forensic needs.

## Key PowerShell Cmdlets for Forensics

- `Get-Process` - List running processes.

- `Get-EventLog` - Retrieve Windows event logs.

- `Get-FileHash` - Generate hashes for file verification.

- `Get-ChildItem` - List files and folders recursively.

- `Get-WmiObject` - Access system information and services.

## Collecting System Information

```sh
Get-ComputerInfo | Out-File "C:\Users\Administrator\Documents\Forensic\SystemInfo.txt"
Get-Process | Format-Table -AutoSize | Out-File "C:\Users\Administrator\Documents\Forensic\ProcessList.txt"
```

## Investigating User Accounts

```sh
# List all AD user accounts
Get-ADUser -Filter * | Select-Object Name, Enabled | Out-File "C:\Users\Administrator\Documents\Forensic\ADUserAccounts.txt"

# List Domain Admins
Get-ADGroupMember -Identity "Domain Admins"

# Get detailed info for a specific user
Get-ADUser -Identity "thierno" -Properties *
```

## Investigating Processes and Services

```sh
# Suspicious processes
Get-WmiObject Win32_Process | Where-Object { $_.ExecutablePath -notlike "C:\Windows\*" } | Out-File "C:\Users\Administrator\Documents\Forensic\SuspiciousProcesses.txt"

# Suspicious services
Get-WmiObject Win32_Service | Where-Object { $_.PathName -notlike "C:\Windows\*" } | Out-File "C:\Users\Administrator\Documents\Forensic\SuspiciousServices.txt"
```

## Scheduled Tasks and Network Connections

```sh
# Suspicious scheduled tasks
Get-ScheduledTask | Where-Object { $_.TaskName -notin @('UpdateTask','SystemTasks','WindowsTasks') } | Out-File "C:\Users\Administrator\Documents\Forensic\SuspiciousScheduledTasks.txt"

# Active network connections
Get-NetTCPConnection | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="Process";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Out-File "C:\Users\Administrator\Documents\Forensic\ActiveConnections.txt"
```

## Registry Forensics (Persistence & Autoruns)

```sh
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run" | Out-File "C:\Users\Administrator\Documents\Forensic\Autoruns.txt"
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" | Out-File "C:\Users\Administrator\Documents\Forensic\AutorunsUser.txt"
```

## Event Logs (Authentication & RDP)

```sh
Get-WinEvent -LogName Security | Where-Object { $_.Id -in 4624,4625,4627 } | Format-Table TimeCreated, Id, Message | Out-File "C:\Users\Administrator\Documents\Forensic\SecurityEvents.txt"
```

- `4624` = Successful login

- `4625` = Failed login (bruteforce, wrong password)

- `4627` = Group membership info during login

## Hashing Suspicious Files

```sh
Get-FileHash "C:\Users\Public\Documents\InfectedFiles\*" -Algorithm SHA256 | Out-File "C:\Users\Administrator\Documents\Forensic\FileHashes.txt"
```

## Simulated Attacks

### Malware Simulation

```sh
$folder = "C:\Users\Public\Documents\InfectedFiles"
New-Item -ItemType Directory -Force -Path $folder

1..5 | ForEach-Object {
    $file = "$folder\File$_.txt"
    Set-Content -Path $file -Value "This is a simulated infected file $_"
    Rename-Item -Path $file -NewName "$file.encrypted"
}

Start-Process -FilePath "notepad.exe"
```

### Unauthorized Access Simulation

```sh
New-LocalUser "intruder" -Password (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) -FullName "Unauthorized User"
Add-LocalGroupMember -Group "Administrators" -Member "intruder"

Write-EventLog -LogName Security -Source "Microsoft-Windows-Security-Auditing" -EventID 4625 -EntryType Failure -Message "Failed login attempt for intruder"
```
