# User Account Activity Monitoring

- Install Sysmon for Linux.

- Route logs to Splunk.

- Simulate malicious activity (suspicious account, scripts, SSH).

- Detect events via Splunk queries.

- Respond to incidents.

## Sysmon for Linux install script

- Save as `install_sysmon.sh` and run:

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

```sh
#!/usr/bin/env bash
set -euo pipefail

wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update
sudo apt-get install -y sysmonforlinux rsyslog

sudo systemctl enable sysmon
sudo systemctl start sysmon
sudo systemctl enable rsyslog
sudo systemctl restart rsyslog

# checks
sysmon -h || true
systemctl status sysmon --no-pager || true
```

#### Run it:

```sh
chmod +x install_sysmon.sh
./install_sysmon.sh
```

## Sysmon configuration (lean & effective)

#### Create `sysmon-config.xml` (inspired by MSTIC, simplified for this lab):

```sh
nano sysmon-config.xml
```

```sh
<?xml version="1.0" encoding="utf-8"?>
<Sysmon schemaversion="4.90">
  <HashAlgorithms>*</HashAlgorithms>
  <EventFiltering>
    <!-- Process creation -->
    <ProcessCreate onmatch="include">
      <CommandLine condition="contains">useradd</CommandLine>
      <CommandLine condition="contains">adduser</CommandLine>
      <CommandLine condition="contains">ssh</CommandLine>
      <CommandLine condition="contains">wget</CommandLine>
      <CommandLine condition="contains">curl</CommandLine>
      <CommandLine condition="contains">nc</CommandLine>
      <CommandLine condition="contains">netcat</CommandLine>
      <CommandLine condition="contains">chmod 777</CommandLine>
      <CommandLine condition="contains">rm -rf</CommandLine>
      <Image condition="end with">/bash</Image>
    </ProcessCreate>

    <!-- File creations/modifications -->
    <FileCreate onmatch="include">
      <TargetFilename condition="end with">.sh</TargetFilename>
      <TargetFilename condition="contains">/tmp/</TargetFilename>
      <TargetFilename condition="end with">.exe</TargetFilename>
    </FileCreate>

    <!-- Network connections (exfiltration / shells) -->
    <NetworkConnect onmatch="include">
      <DestinationPort condition="is">22</DestinationPort>
      <DestinationPort condition="is">4444</DestinationPort>
      <Image condition="end with">/bash</Image>
      <Image condition="end with">/nc</Image>
    </NetworkConnect>

    <!-- Exclude basic noise -->
    <ProcessAccess onmatch="exclude">
      <Image condition="begin with">/usr/bin/sudo</Image>
    </ProcessAccess>
  </EventFiltering>
</Sysmon>
```

#### Apply the configuration:

```sh
sudo sysmon -c sysmon-config.xml
sudo systemctl restart sysmon
```

#### Verify Sysmon logging:

```sh
grep -i sysmon /var/log/syslog | tail -n 50
```

## Route Sysmon Events to Dedicated File

- Create `/etc/rsyslog.d/30-sysmon.conf`:

```sh
# Route events from program "sysmon" to a dedicated file
if ($programname == 'sysmon') then /var/log/sysmon.log
& stop
```

#### Reload rsyslog:

```sh
sudo systemctl restart rsyslog
sudo touch /var/log/sysmon.log && sudo chmod 644 /var/log/sysmon.log
```

#### Check

```sh
tail -f /var/log/sysmon.log
```

## Splunk Universal Forwarder configuration

- Monitor the dedicated file:

```sh
nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

```sh
[monitor:///var/log/sysmon.log]
disabled = false
index = linux_sysmon
sourcetype = sysmon:linux
```

- Props configuration:

```sh
nano /opt/splunkforwarder/etc/system/local/props.conf
```

```sh
[sysmon:linux]
SHOULD_LINEMERGE = false
TIME_PREFIX = timestamp="
TRUNCATE = 0
```

#### Restart the UF:

```sh
sudo /opt/splunkforwarder/bin/splunk restart
```

#### Smoke test in Splunk:

```sh
index=linux_sysmon sourcetype="sysmon:linux" | head 20
```

## Malicious activity simulation script

- Save as `simulate_attack.sh` and run:

```sh
#!/usr/bin/env bash
set -euo pipefail

# Create suspicious user
sudo adduser maluser --disabled-password --gecos ""
echo "maluser:Password123!" | sudo chpasswd

# Drop & execute a suspicious script
cat <<'EOF' | sudo tee /tmp/test.sh >/dev/null
#!/usr/bin/env bash
echo "Malicious script executed"
EOF
sudo chmod +x /tmp/test.sh
/tmp/test.sh

# Local SSH attempt
ssh -o StrictHostKeyChecking=no localhost true || true
```

#### Run and monitor logs:

```sh
chmod +x simulate_attack.sh
./simulate_attack.sh
sudo tail -f /var/log/sysmon.log
```

#### Watch the logs:

```sh
sudo tail -f /var/log/syslog | grep -E "sysmon|maluser" &
sudo tail -f /var/log/auth.log | grep -E "maluser|sshd" &
sudo tail -f /var/log/sysmon.log &
```

## Detection Queries (SPL)

#### Suspicious commands:

```sh
index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*useradd*" OR CommandLine="*adduser*")
index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*wget*" OR CommandLine="*curl*")
index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*nc*" OR CommandLine="*netcat*")
index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*chmod 777*" OR CommandLine="*rm -rf*")
```

#### Files in `/tmp/`

```sh
index=linux_sysmon sourcetype="sysmon:linux" TargetFilename="/tmp/*"
| stats count by TargetFilename, User, Image, CommandLine
```

#### Timeline of events:

```sh
index=linux_sysmon sourcetype="sysmon:linux"
| timechart span=5m count by EventType limit=10
```

## Splunk Alert: Suspicious User Creation

```sh
nano /opt/splunk/etc/apps/search/local/savedsearches.conf
```

```sh
[Linux - Suspicious User Creation]
search = index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*useradd*" OR CommandLine="*adduser*")
cron_schedule = */5 * * * *
alert_type = number of events
number_of_events = 1
relation = greater than
alert.severity = 4
action.email = 1
action.email.to = soc@example.com
action.email.subject = [ALERT] Linux - Suspicious User Creation
```

## Splunk Dashboard (Simple XML)

- Create a new dashboard and paste:

```sh
nano /opt/splunk/etc/apps/search/local/data/ui/views/linux_sysmon_dashboard.xml
```

```sh
<dashboard>
  <label>Linux Sysmon - User Activity Monitoring</label>
  <row>
    <panel>
      <title>Events by type</title>
      <chart>
        <search>
          <query>index=linux_sysmon sourcetype="sysmon:linux" | timechart span=5m count by EventType limit=10</query>
          <earliest>-24h@h</earliest>
          <latest>now</latest>
        </search>
        <option name="charting.chart">line</option>
      </chart>
    </panel>
    <panel>
      <title>User creations (useradd/adduser)</title>
      <table>
        <search>
          <query>index=linux_sysmon sourcetype="sysmon:linux" (CommandLine="*useradd*" OR CommandLine="*adduser*") | table _time, User, CommandLine, Image</query>
          <earliest>-24h@h</earliest>
          <latest>now</latest>
        </search>
      </table>
    </panel>
  </row>
</dashboard>
```

#### Incident Response (IR)

```sh
id maluser
sudo chage -l maluser
grep maluser /etc/passwd
sudo deluser --remove-home maluser
```
