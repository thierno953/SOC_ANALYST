# SSH Brute Force Attack Detection using Wazuh

- Reference: [https://documentation.wazuh.com/current/proof-of-concept-guide/detect-brute-force-attack.html](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-brute-force-attack.html)

#### Enable SSH Service (on target machine, e.g. agent01)

```sh
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl status ssh
```

#### Create User and Password Lists (on attacker machine)

```sh
# nano Users.lst
tom
jerry
donald
msfadmin
user1
bahrain
```

```sh
# nano Passwords.lst
P@ssw0rd
123$
321$
451#
123
```

#### Launch SSH brute-force attack using Medusa

```sh
medusa -U Users.lst -P Passwords.lst -h <Wazuh agent> -M ssh
```

- Optional (verbose output):

```sh
medusa -U Users.lst -P Passwords.lst -h <Wazuh agent> -M ssh -vV
```

- Check if SSH is open:

```sh
nmap -p 22 <Wazuh agent>
```

#### Configure Wazuh Agent to monitor SSH authentication logs (on agent01)

```sh
sudo nano /var/ossec/etc/ossec.conf
```

- Add the following inside the `<ossec_config>` block:

```sh
<!-- SSH logs -->
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

<!-- Optional: Fail2Ban log if used -->
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/fail2ban.log</location>
</localfile>
```

#### Restart Wazuh Agent

```sh
sudo systemctl restart wazuh-agent
```

#### Simulate SSH brute force (attacker machine)

```sh
hydra -L Users.lst -P Passwords.lst <Wazuh agent> ssh
```

#### Expected Result

- After the Medusa attack, Wazuh will detect multiple failed login attempts from the same IP and raise a **Brute Force Attack alert** in the Wazuh dashboard under the `Security Events` section.

- If alerts don’t appear immediately, you can verify with:

```sh
sudo tail -f /var/log/auth.log
sudo tail -f /var/ossec/logs/ossec.log
```

![WAZUH](/Wazuh/assets/08.png)
