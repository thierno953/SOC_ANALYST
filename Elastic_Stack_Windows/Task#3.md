# Monitoring Windows Registry Changes

- Verify ELK and Fleet Agent
- Simulate the attack using Powershell and visualize the alert
- Incident Response

#### Verify ELK

`Registry Editor > Computer`

`Management > Fleet`

`Management > Integrations > integration policies`

#### Simulate the attack and Visualize on ELK Dashboard

```sh
PS C:\Users\Administrator> New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest2" -Value "C:\malwaretest2.exe"
```

- `Analystics > Discover`

```sh
event.code: 13
```
