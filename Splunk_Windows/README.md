#### Installing and setting up Splunk Universal Forwarder on Windows Server 2022

`C:\Program Files > SplunkUniversalForwarder > etc > system > local > inputs.conf`

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

```sh
PS C:\Users\Administrator> cd "C:\Program Files\SplunkUniversalForwarder\bin"
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

#### Analyzing Windows events on Splunk Dashboard

####Search & Reporting

```sh
index="windows_event_logs"
```
