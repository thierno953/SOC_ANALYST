# Task#2: Detect Unusual Network Connection

- Hypothesis: Identify suspicious outbound connections to unknown or malicious IPs.

- Techniques

  - T1071 (Application Layer Protocol)

- Hunting
  - Run a Hunt with artifact Linux.Network.NetstatEnriched
  - Look for Outlier and Establised

`View Artifacts > Linux Only > netstat > Linux.Network.NetstatEnriched > Hunt Artifacts`

![Velociraptor](/Velociraptor-Linux/assets/07.png)
![Velociraptor](/Velociraptor-Linux/assets/08.png)
![Velociraptor](/Velociraptor-Linux/assets/09.png)
![Velociraptor](/Velociraptor-Linux/assets/10.png)

```sh
SELECT *, count() AS Count FROM source(artifact="Linux.Network.NetstatEnriched") WHERE Status="ESTABLISHED"
```
