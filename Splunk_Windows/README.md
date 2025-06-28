# Installation et configuration du Splunk Universal Forwarder sur Windows Server 2022

`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

```sh
[WinEventLog://Application]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:Application

[WinEventLog://Security]
disabled = 0 
index = windows_event_logs
sourcetype = WinEventLog:Security

[WinEventLog://System]
disabled = 0
index = windows_event_logs
sourcetype = WinEventLog:System
```

> Redémarrage du service Splunk

```sh
PS C:\Users\Administrator> cd "C:\Program Files\SplunkUniversalForwarder\bin"
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

#### Analyse des événements Windows dans le tableau de bord Splunk

> `Requêtes dans Search & Reporting`

```sh
index="windows_event_logs"
```
