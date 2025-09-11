# Memory Forensics Cheat Sheet

## Memory forensics:

- Memory forensics is the process of analyzing a computer’s **volatile memory (RAM)** to extract digital artifacts and system activity.

- Provides a **real-time snapshot** of the system, allowing you to:

  - Detect malware, rootkits, and hidden processes

  - Reconstruct user actions

  - Support incident response and legal investigations

- **Importance**: RAM contents disappear when the system shuts down, making it crucial for evidence and investigation.

## Why Memory Forensics Matters

- RAM contains information not found on disk:

  - Running processes and executables

  - Network connections (IP addresses, ports, protocols)

  - Logged-in users and sessions

  - Open files and handles

  - Injected code or malware running in memory

- **Key point for a project**: capturing memory helps reconstruct an attack or user actions **even if traces are deleted from disk**.

## Memory Acquisition Tools

```sh
| OS      | Tools                                      |
| ------- | ------------------------------------------ |
| Windows | FTK Imager, DumpIt, Belkasoft RAM Capturer |
| Linux   | LiME (Linux Memory Extractor)              |
| MacOS   | OSXPmem                                    |
```

- **Best practice**: Always **verify the integrity** of memory dumps using hashes (SHA256, SHA1, MD5) to ensure forensic soundness.

## Installing Volatility

```sh
wget https://github.com/volatilityfoundation/volatility3/archive/refs/tags/v2.4.1.zip
unzip v2.4.1.zip
sudo mv volatility3-2.4.1 /opt/volatility3
cd /opt/volatility3
python3 vol.py --help
```

## Verify memory dump integrity

```sh
# Generate hashes for integrity verification
sha256sum memdump.mem > memdump.mem.sha256
sha1sum memdump.mem > memdump.mem.sha1
md5sum memdump.mem > memdump.mem.md5
```

# Analyzing Memory with Volatility

## Verify & Identify Memory Image

```sh
du -sh memdump.mem
python3 vol.py -f memdump.mem windows.info.Info
```

## Process Analysis

```sh
| Plugin  | Purpose                             | Command                                                                   |
| ------- | ----------------------------------- | ------------------------------------------------------------------------- |
| pslist  | List all running processes          | `vol.py -f memdump.mem windows.pslist.PsList`                             |
| psscan  | Find hidden or terminated processes | `vol.py -f memdump.mem windows.psscan.PsScan`                             |
| pstree  | Display process tree                | `vol.py -f memdump.mem windows.pstree.PsTree`                             |
| cmdline | Show process command-line arguments | `vol.py -f memdump.mem windows.cmdline --pid <PID>`                       |
| dlllist | List DLLs loaded by processes       | `vol.py -f memdump.mem windows.dlllist`                                   |
| handles | Show open handles (files, registry) | `vol.py -f memdump.mem windows.handles`                                   |
| memdump | Dump memory of a specific process   | `vol.py -f memdump.mem windows.memdump --pid 1234 -D ./dumped_processes/` |
```

## Network Analysis

```sh
| Plugin  | Purpose                         | Command                                          |
| ------- | ------------------------------- | ------------------------------------------------ |
| netscan | Scan active network connections | `vol.py -f memdump.mem windows.netscan.NetScan`  |
| netstat | Display network connections     | `vol.py -f memdump.mem windows.netstat.NetStat`  |
| privs   | Show process privileges         | `vol.py -f memdump.mem windows.privileges.Privs` |
```

## Malware & Suspicious Activity

```sh
| Plugin     | Purpose                                         | Command                                               |
| ---------- | ----------------------------------------------- | ----------------------------------------------------- |
| malfind    | Detect injected code / malware                  | `vol.py -f memdump.mem windows.malfind`               |
| apihooks   | Detect API hooks                                | `vol.py -f memdump.mem windows.apihooks.ApiHooks`     |
| modscan    | Scan loaded kernel modules                      | `vol.py -f memdump.mem windows.modscan.ModScan`       |
| ldrmodules | Detect DLL injection                            | `vol.py -f memdump.mem windows.ldrmodules.LdrModules` |
| ssdt       | Scan System Service Descriptor Table (rootkits) | `vol.py -f memdump.mem windows.ssdt.SSDT`             |
| shimcache  | Check executed programs cache                   | `vol.py -f memdump.mem windows.shimcache.ShimCache`   |
| svcscan    | List Windows services                           | `vol.py -f memdump.mem windows.svcscan.SvcScan`       |
| driverscan | Scan loaded kernel drivers                      | `vol.py -f memdump.mem windows.driverscan.DriverScan` |
```

## Autoruns & Persistence

```sh
| Plugin            | Purpose                     | Command                                                                                                 |
| ----------------- | --------------------------- | ------------------------------------------------------------------------------------------------------- |
| autoruns          | Show autorun entries        | `vol.py -f memdump.mem windows.autoruns.Autoruns`                                                       |
| registry.printkey | Check autorun registry keys | `vol.py -f memdump.mem windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"` |
```

## File & Registry Analysis

```sh
| Plugin   | Purpose                            | Command                                           |
| -------- | ---------------------------------- | ------------------------------------------------- |
| filescan | Scan file objects in memory        | `vol.py -f memdump.mem windows.filescan.FileScan` |
| dlllist  | List DLLs loaded by processes      | `vol.py -f memdump.mem windows.dlllist`           |
| handles  | List open handles (files/registry) | `vol.py -f memdump.mem windows.handles`           |
```

## Timeline & Event Analysis

```sh
| Plugin    | Purpose                                | Command                           |
| --------- | -------------------------------------- | --------------------------------- |
| timeliner | Build a timeline of events from memory | `vol.py -f memdump.mem timeliner` |
```

## Useful Resources:

- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)

- [Volatility Framework](https://github.com/volatilityfoundation/volatility3/releases/)

- [VirusTotal](https://www.virustotal.com/gui/home/search)

- [Quickhash](https://www.quickhash-gui.org/)
