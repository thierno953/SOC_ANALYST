# Investigating PowerShell Abuse on Windows Machines

- Verify ELK and Fleet Agent
- Simulate the attack using Powershell and visualize the alert
- Incident Response

`Management > Integrations (search Windows) > Active (Powershell, Powershell Operational, Sysmon Operational, Windows Defender)`

##### Simulate the attack and Visualize on ELK Dashboard

- [Anti Malware Testfile](https://www.eicar.org/download-anti-malware-testfile/)

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

- `Analystics > Discover`

```sh
event.code:11
```
