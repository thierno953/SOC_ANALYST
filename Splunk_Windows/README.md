# Installation and Configuration of Splunk Universal Forwarder on Windows Server 2022

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

#### Restarting the Splunk service

```sh
PS C:\Users\Administrator> cd "C:\Program Files\SplunkUniversalForwarder\bin"
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe status
```

#### Windows Event Log Analysis in Splunk Dashboard

- Search queries in `Search & Reporting`

```sh
index="windows_event_logs"
```

![Enterprise](/Splunk_Windows/assets/splunk_windows_01.png)
