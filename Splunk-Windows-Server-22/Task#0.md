#### 1 - Installing and setting up Splunk Universal Forwarder on Windows Server 2022


```sh
(Program Files --> SplunkUniversalForwarder --> etc --> System --> local)
```

```sh
outputs.conf
```

```sh
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = <IP>:9997

[tcpout-server://<IP>:9997]
```

inputs.conf

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

(Program Files --> SplunkUniversalForwarder --> bin --> splunk)

```sh
PS C:\Users\Administrator> cd "C:\Program Files\SplunkUniversalForwarder\bin"
PS C:\Program Files\SplunkUniversalForwarder\bin> .\splunk.exe restart
```

(Settings --> System --> Server controls --> restart splunk)

#### 2 - Analyzing Windows events on Splunk Dashboard

###### Search & Reporting

```sh
index="windows_event_logs"
```
