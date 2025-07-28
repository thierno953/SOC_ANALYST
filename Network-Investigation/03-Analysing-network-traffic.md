# Network Traffic Analysis

## Identify Top Talkers (Hosts with the Most Traffic)

- **Task 1**: Find the top talkers — hosts generating or receiving the most traffic.

- **Objective**: Detect potential TCP retransmissions and connectivity issues.

![Wireshark](/Network-Investigation/assets/15.png)
![Wireshark](/Network-Investigation/assets/16.png)

## Detect Connection Issues

- **Task 2**: Identify connectivity problems in TCP communication.

- **Objective**: Detect TCP retransmissions and diagnose timeout or ACK issues.

#### TCP Retransmission Scenario:

```sh
1. The client sends a TCP packet to the server:
   Client --------------------> Server
          (Packet sent)

2. The server should respond with an ACK:
   Client <-------------------- Server
          (ACK received)

3. If the client does not receive the ACK in time (timeout), it retransmits the same packet:
   Client --------------------> Server
          (Retransmitted packet)

4. The server eventually responds with an ACK:
   Client <-------------------- Server
          (ACK received)
```

![Wireshark](/Network-Investigation/assets/17.png)
![Wireshark](/Network-Investigation/assets/18.png)
![Wireshark](/Network-Investigation/assets/19.png)
![Wireshark](/Network-Investigation/assets/20.png)

## Analyze ICMP Traffic (Ping)

- **Task 3**: Analyze ICMP Echo Requests and Echo Replies.

- **Objective**: Observe the ping process and check for connectivity or latency issues.

```sh
|----------------------|
|      Request         |
| A ----------------> B|

# ping microsoft.com -4     # IPv4 ping
# ping microsoft.com -t4    # Continuous IPv4 ping (Windows)
```
