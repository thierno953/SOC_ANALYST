# Monitoring Windows Registry Changes

#### Objective

- Detect the creation or modification of keys in the Windows Registry.

- Use Sysmon and ELK to monitor suspicious behavior.

- Respond to a common persistence attempt via the registry key:

  `HKCU:\Software\Microsoft\Windows\CurrentVersion\Run`.

## Verify Integration in ELK

- In Kibana:

- `Management > Fleet > Agents` : Ensure the Fleet Agent is online on the target Windows machine.

- `Management > Integrations > Integration Policies` :

  - Select the **Windows** integration.

  - Ensure **Sysmon Operational** is **enabled**.

  - Make sure that **Sysmon Event ID 13** is being collected.

## Attack Simulation: `Registry Persistence`

- PowerShell command:

```sh
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest1" -Value "C:\malwaretest1.exe"
```

- This command adds a persistence entry to the `Run` key, which is commonly used by malware.

#### Visualization in Kibana

- Interface : `Analytics > Discover`

```sh
event.code:13
```

![ELK](/Elastic_Stack_Windows/assets/04.png)

- **Sysmon Event ID 13**: Activity on a registry key (creation/modification)

## Incident Response

- Response steps:

  - **Verify** the source of the registry key creation.

  - **Remove** the suspicious key if identified as malicious:

```sh
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MalwareTest1"
```
