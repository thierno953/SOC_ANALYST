# Investigating Unauthorized SSH Access Attempts with ELK SIEM

## Check ELK Stack and Fleet Agent

- Access Kibana and verify that the agent is online and collecting system logs:

  - Navigate to:

  `Management > Fleet > Agents`

## Simulate an SSH Brute-Force Attack and View Logs

- From the attack machine (e.g., `attack-ubuntu`):

```sh
# Install Hydra
sudo apt install hydra -y

# Create a password list
echo -e "password\n123456\nadmin\nroot\nthierno" > password.txt

# Launch a brute-force SSH attack against the Fleet Agent
hydra -l thierno -P password.txt <ELASTIC_AGENT_HOST_IP> ssh
```

#### On the Fleet Agent machine:

```sh
# Monitor authentication logs in real time
sudo tail -f /var/log/auth.log
```

- Repeat with another user:

```sh
hydra -l root -P password.txt <ELASTIC_AGENT_HOST_IP> ssh
sudo tail -f /var/log/auth.log
```

#### View SSH Logs in Kibana Discover

- Navigate to: `Analytics > Discover`

- Use **KQL (Kibana Query Language)** to filter SSH failure logs:

```sh
event.outcome: "failure" and process.name: "sshd" and user.name: "thierno"
```

![ELK](/Elastic_Stack_Ubuntu/assets/04.png)

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

- Go to: `Security > Alerts`

- Create an investigation case:

  - Click the alert

  - Click `Take action > Add to new case`

  - Name the case: `SSH brute force attempt`

  - Description: `Attempted SSH brute-force attack`

  - Confirm with: `Create case`

#### Trigger the attack again to generate alerts:

```sh
event.dataset:"system.auth" AND event.outcome:"failure"
```

![ELK](/Elastic_Stack_Ubuntu/assets/05.png)

#### Trigger additional failed login attempts

```sh
for i in {1..5}; do ssh root@<ELASTIC_AGENT_HOST_IP>; done
```

```sh
event.category: authentication AND event.outcome: failure
```

![ELK](/Elastic_Stack_Ubuntu/assets/06.png)
