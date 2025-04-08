# Investigating RDP Brute-Force Attacks on Windows Login

#### Installing Sysmon

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

#### Setting up ELK

`Management > Integrations (search Windows)`

#### Simulate the attack and Visualize on ELK Dashboard

```sh
root@attack-ubuntu:~# apt install hydra -y
root@attack-ubuntu:~# hydra -l administrator -P password.txt <IP_FLEET_AGENT> rdp
```

- `Analystics > Discover`

```sh
winlog.channel:
event.code:3 and source.ip:"<IP_ADDRESS>"
event.code:3 and source.ip:"<IP_ADDRESS>" and rule.name:"RDP"
```
