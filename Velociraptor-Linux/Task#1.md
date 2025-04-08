# Task#1: Suspicous Process Hunting

- Hypothesis: Unauthorized or suspicous processes are running on the system.

- Techniques:

  - T1059.003 - Command and Scripting Interprete
  - T1036 - Masqurerading: Executing Processes from Uncommon Locations

- Hunting
  - Run a Hunt with artifact Linux.Sys.Pslist
  - Look for Outlier
  - Search for any process executed by unwanted commands
  - Look for the file path
  - Reporting

`View Artifacts > Linux Only > Linux.Sys.Pslist > Hunt Artifacts`

![Velociraptor](/Velociraptor-Linux/assets/03.png)
![Velociraptor](/Velociraptor-Linux/assets/04.png)
![Velociraptor](/Velociraptor-Linux/assets/05.png)
![Velociraptor](/Velociraptor-Linux/assets/06.png)

```sh
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime, fqdn FROM source(artifact="Linux.Sys.Pslist")
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime, fqdn, count() AS Count FROM source(artifact="Linux.Sys.Pslist")
SELECT Pid, Name, CommandLine, Exe, Username, CreatedTime, fqdn, count() AS Count FROM source(artifact="Linux.Sys.Pslist") WHERE CommandLine=~"bash"
```

```sh
root@client:~# cd /usr/bin/
root@client:/usr/bin# ps aux | grep Malicious
root@client:/usr/bin# ls -al bash
```
