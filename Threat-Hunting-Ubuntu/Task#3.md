# Detect Unusual Network Connections

### Hypothesis:

- Suspicious outbound network connections to unknown or potentially malicious IP addresses are present on the system.

### Techniques (MITRE ATT&CK Mapping)

- **T1071** – Application Layer Protocol (Adversaries may communicate over application layer protocols to avoid detection.)

### Hunting Approach

- Run a hunt using the `Linux.Network.NetstatEnriched` artifact.

- Identify outliers in active network connections.

- Focus on **ESTABLISHED** connections to detect ongoing communication.

- Review remote IPs and ports for signs of malicious activity.

- Correlate process IDs (PIDs) with their executable paths and commands.

- Report any suspicious findings

### Velociraptor Investigation

- **Navigation Path**:

  - `View Artifacts > Linux Only > netstat > Linux.Network.NetstatEnriched and Linux.Network.Netstat > Hunt Artifacts`

#### Preview Network Data

```sh
/*
Linux.Network.NetstatEnriched
*/
SELECT *
FROM source(artifact="Linux.Network.NetstatEnriched")
LIMIT 5
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/08.png)

#### Count All Network Entries

```sh
SELECT *, count() AS Count
FROM source(artifact="Linux.Network.NetstatEnriched")
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/09.png)

#### Filter for Active ESTABLISHED Connections

```sh
SELECT *, count() AS Count
FROM source(artifact="Linux.Network.NetstatEnriched")
WHERE Status = "ESTABLISHED"
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/10.png)

#### Detailed View of ESTABLISHED Connections

- Sort results by remote address for pattern detection:

```sh
SELECT
  Laddr AS LocalAddress,
  Lport AS LocalPort,
  Raddr AS RemoteAddress,
  Rport AS RemotePort,
  Status,
  Pid,
  ProcInfo,
  Username,
  Fqdn
FROM source(artifact="Linux.Network.NetstatEnriched")
WHERE Status = "ESTABLISHED"
ORDER BY RemoteAddress
LIMIT 100
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/11.png)

### Manual Verification on Endpoint

- Once a suspicious connection is identified, verify on the host:

```sh
# List active established connections
netstat -tunp | grep ESTABLISHED
ss -tunp state established

# Verify the executable linked to the PID
ls -l /proc/<PID>/exe

# Review the process command line
cat /proc/<PID>/cmdline | tr '\0' ' '
```
