# SSH Brute Force Detection and Active Response

- Reference: [https://documentation.wazuh.com/current/user-manual/capabilities/active-response/ar-use-cases/blocking-ssh-brute-force.html](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/ar-use-cases/blocking-ssh-brute-force.html)

## SSH Brute Force Detection + Active Response (Firewall Drop)

#### Simulate SSH brute force (attacker machine)

```sh
hydra -L Users.lst -P Passwords.lst <Wazuh agent> ssh
```

#### Configure Wazuh Manager for Active Response (on Wazuh manager)

- Edit Wazuh configuration:

```sh
sudo nano /var/ossec/etc/ossec.conf
```

- Add these blocks inside the `<ossec_config>` section:

```sh
<!-- Command definition -->
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<!-- Active response -->
<active-response>
  <disabled>no</disabled>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5710</rules_id>   <!-- SSH Brute Force rule ID -->
  <timeout>180</timeout>      <!-- Ban for 3 minutes -->
</active-response>
```

- **Note** You can add more `<rules_id>` entries for additional brute force rules, like `5712`, `5715`, etc.

#### Restart the Wazuh Manager

```sh
sudo systemctl restart wazuh-manager
```

#### Trigger brute force again (attacker machine)

- Try a more aggressive SSH brute force:

```sh
hydra -L Users.lst -P Passwords.lst <Wazuh agent> ssh
```

![WAZUH](/Wazuh/assets/09.png)

- Try manual brute force:

```sh
for i in {1..10}; do ssh wronguser@<Wazuh agent> -p 22; done
```

#### Verify the result

- After a few failed attempts:

  - The attacker IP should be **blocked** by Wazuh's active response.

  - You won’t be able to ping or ssh from the attacking machine:

```sh
ping <Wazuh agent>
```

#### Troubleshooting

- Check Wazuh logs:

```sh
tail -f /var/ossec/logs/ossec.log
```

- View current firewall rules (e.g. iptables or nftables):

```sh
sudo iptables -L -n
```

- Confirm if IP is blocked:

```sh
sudo iptables -L | grep DROP
```
