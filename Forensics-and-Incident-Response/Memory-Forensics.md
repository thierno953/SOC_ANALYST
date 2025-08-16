# Memory Forensics Cheat Sheet

## 1. What is Memory Forensics?

- **Memory forensics** is the process of analyzing a computer’s **volatile memory (RAM)** to extract digital artifacts and system activity.

- Provides a **real-time snapshot** of the system, helping detect malware, uncover rootkits, and reconstruct user actions.

- Essential for **incident response and digital investigations**.

## Why Memory Forensics Matters

- Volatile memory contains key evidence that disappears after shutdown:

  - Running processes & executables

  - Network connections (ports, IPs, protocols)

  - Logged-in users & sessions

  - Open files & handles

  - Injected code and malware running in memory

## Memory Acquisition Tools

- **Windows**: FTK Imager, DumpIt, Belkasoft RAM Capturer

- **Linux**: LiME (Linux Memory Extractor)

- **MacOS**: OSXPmem

## Installing Volatility

```sh
wget https://github.com/volatilityfoundation/volatility3/archive/refs/tags/v2.4.1.zip
unzip v2.4.1.zip
sudo mv volatility3-2.4.1 /opt/volatility3
cd /opt/volatility3
python3 vol.py --help
```

### Hash the dump after acquisition to verify integrity.

```sh
sha256sum memdump.mem > memdump.mem.sha256
sha1sum memdump.mem > memdump.mem.sha1
md5sum memdump.mem > memdump.mem.md5
```

## Analysis Workflow

### Step 1: Verify & Identify Memory Image

```sh
du -sh memdump.mem
sha256sum memdump.mem
sha1sum memdump.mem
md5sum memdump.mem
python3 vol.py -f memdump.mem windows.info.Info
```

### Step 2: Process Analysis

```sh
# List running processes
python3 vol.py -f memdump.mem windows.pslist.PsList

# Scan for hidden processes
python3 vol.py -f memdump.mem windows.psscan.PsScan

# Get command-line arguments of processes
python3 vol.py -f memdump.mem windows.cmdline
python3 vol.py -f memdump.mem windows.cmdline --pid <PID>
```

### Step 3: Network & Connections

```sh
python3 vol.py -f memdump.mem windows.netscan.NetScan
python3 vol.py -f memdump.mem windows.netstat.NetStat
```

### Step 4: Malware & Code Injection

```sh
python3 vol.py -f memdump.mem windows.malfind
python3 vol.py -f memdump.mem windows.dlllist
python3 vol.py -f memdump.mem windows.handles
```

### Step 5: Persistence & Registry

```sh
python3 vol.py -f memdump.mem windows.autoruns
python3 vol.py -f memdump.mem windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"
```

### Step 6: File & Timeline Analysis

```sh
python3 vol.py -f memdump.mem windows.filescan
python3 vol.py -f memdump.mem timeliner
```

### Step 7: Dump Suspicious Artifacts

```sh
python3 vol.py -f memdump.mem windows.memdump --pid <PID> -D ./dumped_processes/
```

## Resources:

- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)

- [Volatility Framework](https://github.com/volatilityfoundation/volatility3/releases/)

- [VirusTotal](https://www.virustotal.com/gui/home/search)

- [Quickhash](https://www.quickhash-gui.org/)
