# Task 3: Monitoring Windows Registry Changes

- Simulate the attack and Visualize the event
- Incident Response

`Start > Registry Editor`

#### Simulate the attack and Visualize on Splunk Dashboard

```sh
PS C:\Users\Administrator> New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"
```

####Search & Reporting

```sh
index="sysmon_logs" sourcetype=XmlWinEventLog source="XmlWinEventLog:Sysmon" "*malwaretest*"
```

#### Incident Response

```sh
PS C:\Users\Administrator> Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"
```
