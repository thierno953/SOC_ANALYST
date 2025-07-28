# Wireshark - Network Analysis Tool

- Wireshark is a powerful and widely-used network protocol analyzer. It allows users to capture, inspect, and analyze network packets in real time. It is an essential tool for network administrators, cybersecurity analysts, and anyone investigating network behavior or issues.

## Network Investigation with Wireshark

- **Capture and Display Filters**

  - **Capture filters**: Apply before starting a capture to restrict which packets Wireshark records (e.g., capture only traffic on port 80).

  - **Display filters**: Apply after or during a capture to filter the visible packets (e.g., show only DNS traffic).

![Wireshark](/Network-Investigation/assets/0.png)
![Wireshark](/Network-Investigation/assets/00.png)

## Wireshark Profiles and Interface

- Wireshark's interface is divided into three main panels:

#### Packet List (Top Panel)

- Shows all captured packets, including:

  - Timestamp

  - Source and Destination addresses

  - Protocol used

  - Brief description of the content

#### Packet Details (Middle Panel)

- Displays OSI layer information of the selected packet:

  - **Layer 1 (Physical)** – Frame info, size

  - **Layer 2 (Data Link)** – MAC addresses

  - **Layer 3 (Network)** – IP addresses

  - **Layer 4 (Transport)** – TCP/UDP ports

  - **Higher layers** – HTTP, DNS, TLS, etc.

#### Packet Bytes (Bottom Panel)

- Shows the selected packet in **raw hexadecimal and ASCII form**.

![Wireshark](/Network-Investigation/assets/01.png)
![Wireshark](/Network-Investigation/assets/02.png)
![Wireshark](/Network-Investigation/assets/03.png)
![Wireshark](/Network-Investigation/assets/04.png)
![Wireshark](/Network-Investigation/assets/05.png)

## Colorizing Traffic

- Wireshark uses default coloring rules to help quickly identify different types of traffic:

  - **Green**: TCP traffic

  - **Blue**: DNS, HTTP

  - **Black**: Packet loss or errors

  - **Light** Purple: ICMP

- You can customize these rules in the "View > Coloring Rules" menu.

![Wireshark](/Network-Investigation/assets/06.png)
![Wireshark](/Network-Investigation/assets/07.png)
![Wireshark](/Network-Investigation/assets/08.png)

## TCP and UDP Port

- Wireshark helps visualize and analyze communications using TCP and UDP protocols.

  - **TCP (Transmission Control Protocol)** is connection-oriented (e.g., HTTP, HTTPS, FTP)

  - **UDP (User Datagram Protocol)** is connectionless (e.g., DNS, SNMP, DHCP)

#### Common Ports:

```sh
| Protocol | Port | Description           |
| -------- | ---- | --------------------- |
| HTTP     | 80   | Web traffic           |
| HTTPS    | 443  | Encrypted web traffic |
| DNS      | 53   | Domain name service   |
| FTP      | 21   | File Transfer         |
| SSH      | 22   | Secure Shell          |
```

![Wireshark](/Network-Investigation/assets/10.png)
![Wireshark](/Network-Investigation/assets/11.png)
![Wireshark](/Network-Investigation/assets/12.png)
![Wireshark](/Network-Investigation/assets/13.png)
![Wireshark](/Network-Investigation/assets/14.png)
