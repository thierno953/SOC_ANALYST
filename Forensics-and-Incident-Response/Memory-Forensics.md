# Memory Forensics Cheat Sheet

## What is Memory Forensics?

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
# Generate hashes for integrity verification
sha256sum memdump.mem > memdump.mem.sha256
sha1sum memdump.mem > memdump.mem.sha1
md5sum memdump.mem > memdump.mem.md5
```

## Volatility Plugins Cheat Sheet

### Verify & Identify Memory Image

```sh
du -sh memdump.mem
python3 vol.py -f memdump.mem windows.info.Info
```

### Process Analysis

```sh
| Plugin    | Description                             | Command                                                                           |
| --------- | --------------------------------------- | --------------------------------------------------------------------------------- |
| `pslist`  | Lists running processes                 | `python3 vol.py -f memdump.mem windows.pslist.PsList`                             |
| `psscan`  | Scan for hidden or terminated processes | `python3 vol.py -f memdump.mem windows.psscan.PsScan`                             |
| `pstree`  | Displays process tree                   | `python3 vol.py -f memdump.mem windows.pstree.PsTree`                             |
| `cmdline` | Shows command line arguments            | `python3 vol.py -f memdump.mem windows.cmdline --pid <PID>`                       |
| `dlllist` | Lists DLLs loaded by processes          | `python3 vol.py -f memdump.mem windows.dlllist`                                   |
| `handles` | Shows open handles by processes         | `python3 vol.py -f memdump.mem windows.handles`                                   |
| `memdump` | Dump memory of a specific process       | `python3 vol.py -f memdump.mem windows.memdump --pid 1234 -D ./dumped_processes/` |
```

### Network Analysis

```sh
| Plugin    | Description                     | Command                                                  |
| --------- | ------------------------------- | -------------------------------------------------------- |
| `netscan` | Scan active network connections | `python3 vol.py -f memdump.mem windows.netscan.NetScan`  |
| `netstat` | Display network connections     | `python3 vol.py -f memdump.mem windows.netstat.NetStat`  |
| `privs`   | Show process privileges         | `python3 vol.py -f memdump.mem windows.privileges.Privs` |
```

### Malware & Suspicious Activity

```sh
| Plugin       | Description                                       | Command                                                       |
| ------------ | ------------------------------------------------- | ------------------------------------------------------------- |
| `malfind`    | Detect injected code & hidden malware             | `python3 vol.py -f memdump.mem windows.malfind`               |
| `apihooks`   | Detect API hooks                                  | `python3 vol.py -f memdump.mem windows.apihooks.ApiHooks`     |
| `modscan`    | Scan loaded kernel modules                        | `python3 vol.py -f memdump.mem windows.modscan.ModScan`       |
| `ldrmodules` | Detect DLL injection                              | `python3 vol.py -f memdump.mem windows.ldrmodules.LdrModules` |
| `ssdt`       | Scan System Service Descriptor Table for rootkits | `python3 vol.py -f memdump.mem windows.ssdt.SSDT`             |
| `shimcache`  | Check AppCompat cache for executed programs       | `python3 vol.py -f memdump.mem windows.shimcache.ShimCache`   |
| `svcscan`    | List Windows services                             | `python3 vol.py -f memdump.mem windows.svcscan.SvcScan`       |
| `driverscan` | Scan for loaded kernel drivers                    | `python3 vol.py -f memdump.mem windows.driverscan.DriverScan` |
```

### Autoruns & Persistence

```sh
| Plugin              | Description                 | Command                                                                                                         |
| ------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `autoruns`          | Shows autorun entries       | `python3 vol.py -f memdump.mem windows.autoruns.Autoruns`                                                       |
| `registry.printkey` | Check registry autorun keys | `python3 vol.py -f memdump.mem windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"` |
```

### File & Registry Analysis

```sh
| Plugin     | Description                                    | Command                                                   |
| ---------- | ---------------------------------------------- | --------------------------------------------------------- |
| `filescan` | Scan for file objects in memory                | `python3 vol.py -f memdump.mem windows.filescan.FileScan` |
| `dlllist`  | List DLLs loaded by processes                  | `python3 vol.py -f memdump.mem windows.dlllist`           |
| `handles`  | List open handles (files, registry keys, etc.) | `python3 vol.py -f memdump.mem windows.handles`           |
```

### Timeline & Event Analysis

```sh
| Plugin      | Description                          | Command                                   |
| ----------- | ------------------------------------ | ----------------------------------------- |
| `timeliner` | Build timeline of events from memory | `python3 vol.py -f memdump.mem timeliner` |
```

## Resources:

- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)

- [Volatility Framework](https://github.com/volatilityfoundation/volatility3/releases/)

- [VirusTotal](https://www.virustotal.com/gui/home/search)

- [Quickhash](https://www.quickhash-gui.org/)
