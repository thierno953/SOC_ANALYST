# Investigating PowerShell Abuse on Windows Machines

#### Objective

- Verify ELK and Fleet Agent setup

- Simulate the attack using PowerShell and visualize the alert

- Perform incident response

## Verify Log Integration in ELK

- In the Kibana interface:
- Navigate to:

`Management > Integrations > Windows`

- Ensure the following sources are **enabled**:

  - Forwarded

  - PowerShell

  - PowerShell Operational 

  - Sysmon Operational

  - Windows Defender

- These logs must be collected by the Elastic Agent installed on the target machine.

## PowerShell Attack Simulation

- PowerShell command (to be run from an Administrator session)[Anti Malware Testfile](https://www.eicar.org/download-anti-malware-testfile/)

```sh
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "$env:USERPROFILE\Downloads\eicar.com.txt"
```

- This command downloads an antivirus test file (harmless but designed to trigger detection engines)

## Visualization in Kibana (ELK)

- Interface: `Analytics > Discover`

#### Useful Query

```sh
event.code:11
```

- **Sysmon Event ID 11**: File creation detection, used to observe the creation of potentially malicious files.
