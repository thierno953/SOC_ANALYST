# Investigation of RDP Brute-Force Attacks on Windows Connections

## Objective

- Detect a brute-force attack via RDP on a Windows host.

- Use **Sysmon** to log the connections.

- Use the **ELK Stack** to visualize the events.

## Installing Sysmon

- Download Sysmon and the configuration file:

  - [Sysmon (Microsoft)](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

  - [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)

#### Recommended location: `C:\Sysmon\`

- PowerShell Commands:

```sh
# Navigate to the Sysmon directory
cd C:\Sysmon\

# Install Sysmon with the configuration file
.\Sysmon64.exe -i .\sysmonconfig-export.xml -accepteula

# Check the service status
Get-Service Sysmon*

# Update configuration
.\Sysmon64.exe -c .\sysmonconfig-export.xml

# Restart with updated configuration
.\Sysmon64.exe -u .\sysmonconfig-export.xml
```

#### Log File Location Path: `Event Viewer > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

![ELK](/Elastic_Stack_Windows/assets/01.png)

## Integration with ELK (Fleet/Elastic Agent)

- In the Kibana interface Go to: `Management > Integrations > Windows`

- Ensure the following sources are **enabled**:

  - Forwarded

  - PowerShell

  - PowerShell Operational

  - Sysmon Operational

- These logs must be collected by the Elastic Agent installed on the target machine.

## RDP Attack Simulation Attacking machine (Ubuntu):

```sh
# Install hydra
apt install hydra -y

# Launch a brute-force RDP attack
hydra -l administrator -P password.txt rdp://<target-ip>
```

![ELK](/Elastic_Stack_Windows/assets/02.png)

#### Visualization in Kibana (ELK)

- Interface: `Analytics > Discover` use the following filters:

```sh
event.code: 3 and source.ip: "<IP_ATTACKER>" and rule.name: "RDP"
```

![ELK](/Elastic_Stack_Windows/assets/03.png)
