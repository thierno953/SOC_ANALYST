# Memory Forensics

- Memory forensics is the process of analyzing a computer's volatile memory (RAM) to extract critical information, artifacts, and insights related to system activities, security incidents, and digital investigations.

- It provides a real-time snapshot of the system's state, enabling the detection of malware, uncovering of rootkits, and reconstruction of user activities for incident response and digital forensics.

## Benefits of Memory Forensics

- This is useful because of the way in which processes, files and programs are run in memory and once a snapshot has been captured, many important facts can be ascertained by the investigator, such as:

  - Processes running

  - Executable files that are running

  - Open ports, IP addresses, and other networking information

  - Users that are logged into the system, and from where

  - Files that are open and by whom.

## Volatile Data

- In memory forensics, volatile data refers to information stored in a computer's volatile memory (RAM or Random Access Memory) that is temporary and is lost when the system is powered off or restarted.

- Volatile data in contrasts to non-volatile data, which is stored on persistent storage devices like hard drives and remains intact even after a system shutdown.

- Volatile data is something that any incident responder needs to be aware of, the reason being is that when dealing with a compromised device one of the first reactions may be to turn the device off to contain the threat.

- The problem with this approach is that any malware that is running on the device will be running in memory. So any network connections and running processes will be lost, this is because the malware has been running in memory and this data is now lost.

## Memory Dump

- A memory dump or RAM dump is a snapshot of memory that has been captured for memory analysis. When a RAM dump is captured it will contain data relating to any running processes at the time the capture was taken.

- Memory Acquisition with FTK Imager [https://archive.org/download/access-data-ftk-imager-4.7.1.](https://archive.org/download/access-data-ftk-imager-4.7.1)

## Volatility

- **Volatility** is an open-source memory forensics framework used for analysing volatile memory (RAM) in digital investigations.

- **Purpose**: It helps extract digital artifacts such as running processes, network connections, and loaded drivers from memory dumps.

- **Features**: Offers a wide range of plugins for analyzing memory images from various operating systems (Windows, Linux, macOS, etc.).

- **Community**: Volatility has a large community of contributors and users who actively develop plugins, share knowlodge, and provide support.

## Key Components of Volatility

- **Volatility Core**: Provides the foundation for memory analysis, including memory address translation, process listing, and plugin management.

- **Plugins**: Specialized tools that extract specific information from memory images, such as pslist for listing processes and connections, netscan for network analysis, and filescan for file system exploration.

## Installing Volatility

- Official site: [https://volatilityfoundation.org/the-volatility-framework/.](https://volatilityfoundation.org/the-volatility-framework/)

- Latest release: [https://github.com/volatilityfoundation/volatility3/releases/tag/v2.4.1](https://github.com/volatilityfoundation/volatility3/releases/tag/v2.4.1)

```sh
wget https://github.com/volatilityfoundation/volatility3/archive/refs/tags/v2.4.1.zip
unzip v2.4.1.zip
sudo mv volatility3-2.4.1 /opt/volatility3
cd /opt/volatility3
python3 vol.py --help
```

## Memory Analysis Commands

#### Retrieve memory image information

```sh
python3 vol.py -f memdump.mem windows.info
```

#### List running processes

```sh
python3 vol.py -f memdump.mem windows.pslist
```

#### Scan for hidden processes

```sh
python3 vol.py -f memdump.mem windows.psscan.PsScan
```

#### Get command line arguments of processes

```sh
python3 vol.py -f memdump.mem windows.cmdline
python3 vol.py -f memdump.mem windows.cmdline --pid 2580
```

#### Network scan (for active connections)

```sh
python3 vol.py -f memdump.mem windows.netscan
```

#### Additional Resource: `VirusTotal`

- Analyse de fichiers ou de hachages suspects: [https://www.virustotal.com/gui/home/search.](https://www.virustotal.com/gui/home/search)
