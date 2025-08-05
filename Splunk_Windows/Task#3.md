# Investigation of Suspicious Windows Registry Modifications

- Preliminary Checks

  - **Verify that Sysmon is installed** with a configuration that collects Registry-related events (Event IDs 12, 13, and 14).

  - **Confirm that Splunk Universal Forwarder is properly collecting Sysmon logs** (`index=sysmon_logs`).

  - Open the Registry Editor: `Start > Registry Editor`

## Attack Simulation: Malicious Registry Modification

#### Simulate a malicious registry modification:

```sh
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"

# Verify the creation
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" | Select-Object MalwareTest
echo 'echo malwaretest' > C:\malwaretest.exe
```

#### Search & Reporting Query in Splunk

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*malwaretest*"
```

## Incident Response: Delete Malicious Registry Key

```sh
# Remove the malicious key
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"

# Confirm the removal
Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```
