# Surveillance des activités des comptes utilisateurs avec Sysmon pour Linux

- **Objectif** : Surveiller les activités des comptes utilisateurs à l'aide de Sysmon pour Linux et visualiser les résultats dans Splunk.
- **Étapes** :
  - Installer Sysmon pour Linux.
  - Configurer Splunk pour surveiller les logs Sysmon.
  - Simuler des activités de compte utilisateur.
  - Visualiser les résultats dans Splunk.
  - Répondre à l'incident.

#### 1 - Installation de Sysmon pour Linux

- Mettre à jour les paquets

```sh
root@forwarder:~# apt-get update
```

- Télécharger et installer le package Microsoft

```sh
root@forwarder:~# curl -s https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -o packages-microsoft-prod.deb
root@forwarder:~# sudo dpkg -i packages-microsoft-prod.deb
```

- Mettre à jour les paquets après l'ajout du référentiel

```sh
root@forwarder:~# sudo apt-get update
```

- Installer Sysmon pour Linux

```sh
root@forwarder:~# sudo apt-get install sysmonforlinux
```

- Créer un fichier de configuration Sysmon

```sh
root@forwarder:~# nano sysmon-config.xml
```

- Coller le contenu suivant dans sysmon-config.xml

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
        <Rule name="TechniqueID=T1105,TechniqueName=Ingress Tool Transfer - Ncat" groupRelation="and">
            <Image condition="end with">ncat</Image>
            <Image condition="end with">nc</Image>
            <CommandLine condition="contains">-e</CommandLine> <!-- Détecte l'option de reverse shell -->
            <CommandLine condition="contains any">/bin/bash;/bin/sh;/bin/dash</CommandLine> <!-- Exécution de shell -->
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

- Installer la configuration Sysmon

```sh
root@forwarder:~# sysmon -i sysmon-config.xml
```

- Redémarrer le service Sysmon

```sh
root@forwarder:~# systemctl restart sysmon
```

- Vérifier le statut de Sysmon

```sh
root@forwarder:~# systemctl status sysmon
```

- Vérifier les logs Sysmon dans syslog

```sh
root@forwarder:~# cd /var/log/
root@forwarder:/var/log# grep -i sysmon syslog
```

#### 2 - Configuration de Splunk pour surveiller les logs Sysmon

- Éditer le fichier de configuration des inputs de Splunk

```sh
root@forwarder:~# nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

- Ajouter ou modifier les sections suivantes

```sh
[monitor:///var/log/syslog]
disabled = false
index = linux_os_logs
sourcetype = syslog

#[monitor:///var/log/audit/audit.log]
#disabled = false
#sourcetype = auditd
#index = linux_file_integrity

#[monitor:///var/log/suricata/eve.json]
#disabled = false
#index = network_security_logs
#sourcetype = suricata
```

- Redémarrer Splunk pour appliquer les changements

```sh
root@forwarder:~# /opt/splunkforwarder/bin/splunk restart
```

![splunk](/Splunk-Ubuntu-Server-22/assets/12.png)

###### Search & Reporting

```sh
index="linux_os_logs"
index="linux_os_logs" sysmon
index="linux_os_logs" process=sysmon
```

#### 3 - Simulation d'activités de compte utilisateur et visualisation dans Splunk

- Créer un utilisateur suspect

```sh
root@forwarder:~# adduser maluser
Adding user `maluser' ...
Adding new group `maluser' (1004) ...
Adding new user `maluser' (1003) with group `maluser' ...
The home directory `/home/maluser' already exists.  Not copying from `/etc/skel'.
New password:
Retype new password:
No password has been supplied.
New password:
Retype new password:
No password has been supplied.
New password:
Retype new password:
No password has been supplied.
passwd: Authentication token manipulation error
passwd: password unchanged
Try again? [y/N]
Changing the user information for maluser
Enter the new value, or press ENTER for the default
        Full Name []: malicious user
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n]
root@forwarder:~#
```

- Surveiller les logs en temps réel

```sh
root@forwarder:~# tail -f /var/log/syslog | grep maluser
```

- Recherche et reporting dans Splunk

```sh
index="linux_os_logs" process=sysmon maluser
```

- Supprimer l'utilisateur suspect

```sh
root@forwarder:~# deluser maluser
```

#### 4 - Réponse à l'incident

- Créer un utilisateur suspect pour tester la réponse

```sh
root@forwarder:~# adduser maluser
Adding user `maluser' ...
Adding new group `maluser' (1004) ...
Adding new user `maluser' (1003) with group `maluser' ...
The home directory `/home/maluser' already exists.  Not copying from `/etc/skel'.
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for maluser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
root@forwarder:~#
```

- Vérifier les informations de l'utilisateur

```sh
root@forwarder:~# less /etc/passwd
root@forwarder:~# chage -l maluser
```

- Supprimer l'utilisateur suspect

```sh
root@forwarder:~# deluser maluser
```
