# Investigating Unauthorized SSH Access Attempts with ELK SIEM

## Check ELK Stack and Fleet Agent

- Access Kibana and verify that the agent is online and collecting system logs:

  - Navigate to:

  `Management > Fleet > Fleet Agent > View more agent metrics > Agent Info`

## Simulate an SSH Brute-Force Attack and View Logs

- From the attack machine (e.g., `attack-ubuntu`):

```sh
# Install Hydra
sudo apt install hydra -y

# Create a password list
echo -e "password\n123456\nadmin\nroot\nthierno" > password.txt

# Launch a brute-force SSH attack against the Fleet Agent
hydra -l thierno -P password.txt <FLEET_AGENT_IP> ssh
```

#### On the Fleet Agent machine:

```sh
# Monitor authentication logs in real time
tail -f /var/log/auth.log
```

- Repeat with another user:

```sh
hydra -l root -P password.txt <FLEET_AGENT_IP> ssh
tail -f /var/log/auth.log
```

#### View SSH Logs in Kibana Discover

- Navigate to: `Analytics > Discover`

- Use **KQL (Kibana Query Language)** to filter SSH failure logs:

```sh
event.outcome: "failure" and process.name: "sshd" and user.name: "thierno"
```

## Create Elastic SIEM Detection Rules

- Go to:` Security > Rules > Detection Rules > Add Elastic rules`

- Add existing Elastic rules:

  - Click **Add Elastic rules**

  - Search for: `ssh brute`

  - Install and **enable** relevant rules such as:

    - **Potential Successful SSH Brute Force Attack**

    - **Potential External Linux SSH Brute Force**

    - **Potential Internal Linux SSH Brute Force**

## Generate and Manage a Security Alert

- Trigger the attack again to generate alerts:

```sh
hydra -l thierno -P password.txt <FLEET_AGENT_IP> ssh
hydra -l root -P password.txt <FLEET_AGENT_IP> ssh

# Trigger additional failed login attempts
for i in {1..5}; do ssh fleet-server@<IP>; done
```

- Go to: `Security > Alerts`

- Create an investigation case:

  - Click the alert

  - Click `Take action > Add to new case`

  - Name the case: `SSH brute force attempt`

  - Description: `Attempted SSH brute-force attack`

  - Confirm with: `Create case`
