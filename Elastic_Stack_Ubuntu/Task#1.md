# Task#1 Investigating Unauthorized SSH Access Attempts Using ELK SIEM

- Verify the ELK and Agent
- Simulate the Attack and visualize the events
- Create Elastic Security rules, detect and Investigate
- Incident Response

#### Verify the ELK and Fleet Agent

`Management > Fleet > Fleet-Agent > View more agent metrics`

#### Simulate the attack and Visualize the events

```sh
root@attack-ubuntu:~# sudo apt install hydra
root@attack-ubuntu:~# echo -e "password\n123456\nadmin\nroot\nthierno" > password.txt
root@attack-ubuntu:~# hydra -l thierno -P password.txt <IP_FLEET_AGENT> ssh
```

```sh
root@fleet-agent:~# tail -f /var/log/auth.log
```

```sh
root@attack-ubuntu:~# hydra -l root -P password.txt <IP_FLEET_AGENT> ssh
```

```sh
root@fleet-agent:~# tail -f /var/log/auth.log
```

`Analystics > Discover`

```sh
event.outcome: "failure" and process.name: "sshd"
event.outcome: "failure" and process.name: "sshd" and user.name: thierno
```

#### Create Elastic Security rules, detect and Investigate

`Security > Rules > Detection rules (SIEM) > Add Elastic rules > search > ssh brute and active Potential Successful, External, Internal`

#### Attack Machine

```sh
root@attack-ubuntu:~# hydra -l thierno -P password.txt <IP_FLEET_AGENT> ssh
root@attack-ubuntu:~# hydra -l root -P password.txt <IP_FLEET_AGENT> ssh
```

`Security > Alerts > Take action > add to new case [Name: SSH brute force attempt; Description: brute force and Create case]`
