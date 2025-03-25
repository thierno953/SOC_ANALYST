# Memory Forensics

- Memory Forensics is the process of analyzing a computer's volatile memory (RAM) to extract critical information, artifacts, and insights related to system activities, security incidents, and digital investigations.
- It provides a real-time snapshot of a system's state, enabling the detection of malware, uncovering rootkits, and reconstructing user activities for incident response and digital forensics.

#### Benefits of Memory Forensics

- This is useful because of the way in which processes, files and programs are run in memory, and once a snapshot has been captured, many important facts can be ascertained by the investigator, such as:

- Processes running
- Executable files that are running
- Open ports, IP addresses and other networking information
- Users that are logged into the system, and from where
- Files that are open and by whom

#### Volatile Data

- In memory forensics, volatile data refers to information stored in a computer's volatile memory (RAM or Random Access Memory) that is temporary and is lost when the system is powered off or restarted.
- Volatile Data is in constrat to non-volatile data, which is stored on persistent storage devices like hard drives and remains intact even after a system shutdown.
- Volatile Data is something that any incident responder needs to be aware of the reason being is that when dealing with a compromised device one of the first reactions may be to turn the device off to contain the threat.
- The problem with this approach is that any malware that is running on the device will be running in memory. So any network connections and running processes will be lost, this is because the malware has been running in memory and this data is now lost.

#### Memory Dump

- A memory dump or RAM dump i a snapshot of memory that has been captured for memory analysis. When a RAM dump is captured it will contain data relating to any running processes at the time the capture was taken.

#### Acquiring memory with FTK Image

- [https://archive.org/download/access-data-ftk-imager-4.7.1.](https://archive.org/download/access-data-ftk-imager-4.7.1)

#### Volatility

- Volatility is an open-source memory forensics framework used for analyzing volatile memory (RAM) in digital investigations.
- Purpose: It helps extract digital artifacts such as running processes, network connections, and loaded drivers from memory dumps.
- Features: Offers a wide range of plugins for analyzing memory images from various operating systems (Windows, Linux, MacOS, etc.).
- Community: Volatility has a large community of contributors and users who actively develop plugins, share knowledge, and provide support.

#### Key Components of Volatility

- Volatility Core: Provides the foundation for memory analysis, including memory address translation, process listing, and plugin management.
- Plugins: Specialized tools that extract specific information from memory images, such as pslist for listing processes and connections, netscan for network analysis, and filescan for file system exploration.

#### Installing Volatility

- [https://volatilityfoundation.org/the-volatility-framework/.](https://volatilityfoundation.org/the-volatility-framework/)

- [https://github.com/volatilityfoundation/volatility3/releases/tag/v2.5.0](https://github.com/volatilityfoundation/volatility3/releases/tag/v2.5.0)

```sh
uzip volatility3-2.5.0
cd /opt/
cd volatility/volatility3-2.5.0/
python3 vol.py --help
python3 vol.py -f memdump.mem windows.vadinfo.VadInfo
```

#### 1 - Retrieve Image Information

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.info
```

#### 2 - Collect Process Information

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.pslist
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.psscan.PsScan
```

#### 3 - Retrieving Command Line

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.cmdline
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.cmdline --pid 2580
```

#### 4 - Collecting Network Information

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.netscan
```

- [https://www.virustotal.com/gui/home/search.](https://www.virustotal.com/gui/home/search)
