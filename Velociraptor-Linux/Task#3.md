# Task#3: Detect Unauthorized Package Installations

- Hypothesis: Adversaries install rogue package for backdoors or privilege escalation.

- Techniques

  - T1203 (Exploitation for Client Execution)

- Hunting
  - Run a Hunt with artifact Linux.Debian.Packages
  - Look for unauthorized packages

`View Artifacts > Linux Only > Linux.Debian.Packages > Hunt Artifacts`

![Velociraptor](/Velociraptor-Linux/assets/11.png)

```sh
/*
## Combined results
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
    WHERE Package = "nmap"
  },
  snaps={
    SELECT Name,
           'snap' AS Type,
           InstalledSize,
           Version,
           _Description,
           NULL AS Architecture
    FROM source(artifact="Linux.Debian.Packages/Snaps")
    WHERE Package = "nmap"
  })
```
