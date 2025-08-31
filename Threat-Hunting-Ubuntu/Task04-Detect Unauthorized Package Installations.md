# Detect Unauthorized Package Installations

### Hypothesis:

- Adversaries may install unauthorized packages to establish backdoors, enable privilege escalation, or facilitate post-exploitation activities..

### Techniques (MITRE ATT&CK Mapping)

- **T1203** - Exploitation for Client Execution

### Hunting Approach

- Run a hunt using the `Linux.Debian.Packages` artifact.

- Search for packages commonly used in attacks (e.g., network scanners, password crackers, reverse shell tools).

- Review package descriptions, versions, and installation sizes.

- Document any findings and verify their legitimacy with system owners.

### Velociraptor Investigation

- **Navigation Path**:
  - `View Artifacts > Linux Only > Linux.Debian.Packages > Hunt Artifacts`

### Combined Search for Deb and Snap Packages

- This query checks both .deb and snap packages for suspicious tools such as **nmap, netcat, telnet, john, and hydra**:

```sh
/*
Combined results from Debian and Snap packages
*/
LET ColumnTypes <= dict(`_Description`='nobreak')

SELECT *
FROM chain(
  debs={
    SELECT Package AS Name,
           'deb' AS Type,
           InstalledSize,
           Version,
           _Description,
           Architecture
    FROM source(artifact="Linux.Debian.Packages/DebPackages")
    WHERE Package IN ("nmap", "netcat", "telnet", "john", "hydra")
  },
  snaps={
    SELECT Name,
           'snap' AS Type,
           InstalledSize,
           Version,
           _Description,
           NULL AS Architecture
    FROM source(artifact="Linux.Debian.Packages/Snaps")
    WHERE Package IN ("nmap", "netcat", "telnet", "john", "hydra")
  })
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/12.png)

### Analysis Tips

- Nmap, netcat, and telnet are often used for network reconnaissance and pivoting.

- John the Ripper and Hydra are password-cracking tools commonly used in brute-force attacks.

- If found on production servers without business justification, these packages should be considered suspicious.

- Check installation timestamps (via system logs) to correlate with other suspicious activity.

### Manual Verification

- If a suspicious package is detected, verify directly on the endpoint:

```sh
# Check if the package is installed
dpkg -l | grep -E 'nmap|netcat|telnet|john|hydra'

# For snap packages
snap list | grep -E 'nmap|netcat|telnet|john|hydra'

# Check installation date
grep "install " /var/log/dpkg.log
```
