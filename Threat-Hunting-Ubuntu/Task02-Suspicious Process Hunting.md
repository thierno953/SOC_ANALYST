# Suspicious Process Hunting

### Hypothesis:

- Unauthorized or suspicious processes are running on the system.

### Techniques (MITRE ATT&CK Mapping)

- **T1059.003** - Command and Scripting Interpreter: Unix Shell

- **T1036** - Masquerading: Executing Processes from Uncommon Locations

### Hunting Approach

- Run a hunt using the `Linux.Sys.Pslist` artifact.

- Identify outlier processes.

- Look for processes executed by unexpected or unauthorized commands.

- Review the executable file paths.

- Document findings in a report.

### Velociraptor Investigation

- **Navigation Path**:

  - `View Artifacts > Linux Only > Linux.Sys.Pslist > Hunt Artifacts`

#### Basic Process Enumeration

```sh
/*
Linux.Sys.Pslist
*/
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime, fqdn
FROM source(artifact="Linux.Sys.Pslist")
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/04.png)

#### Group by Process Name

- Identify the most frequently running processes:

```sh
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime, fqdn, count() AS count
FROM source(artifact="Linux.Sys.Pslist")
GROUP BY Name
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/05.png)

#### Search for Processes Using Bash

- Suspicious use of bash in non-interactive contexts can be a sign of malicious activity:

```sh
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime
FROM source(artifact="Linux.Sys.Pslist")
WHERE CommandLine =~ "bash"
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/06.png)

#### Identify Executables in Uncommon Locations

- Masquerading techniques often involve binaries outside standard directories:

```sh
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime
FROM source(artifact="Linux.Sys.Pslist")
WHERE NOT Exe =~ "^/usr/bin/" AND NOT Exe =~ "^/bin/"
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/07.png)

### Manual Verification

- If a suspicious process is detected, verify directly on the endpoint:

```sh
# Check if a process named "malicious" is running
ps aux | grep -i malicious

# Check the exact location and permissions of bash
ls -al /usr/bin/bash
ls -al $(which bash)
```
