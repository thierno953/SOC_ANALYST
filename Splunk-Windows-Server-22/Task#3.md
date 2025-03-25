# Task 3: Monitoring Windows Registry Changes

- Objective
- Sysmon(If not installed)
- Set up Splunk(if needed)
- Simulate the attack and visualize the event
- Incident Response

# 1 - Simulate the attack and Visualize on Splunk Dashboard

```sh
PS C:\Users\Administrator> New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest" -Value "C:\malwaretest.exe"
```

###### Search & Reporting

```sh
index="sysmon_logs" sourcetype="XmlWinEventLog:Sysmon" "*malwaretest*"
```

# 2 - Incident Response

```sh
PS C:\Users\Administrator> Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest"
```
