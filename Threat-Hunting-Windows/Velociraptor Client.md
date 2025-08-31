# Velociraptor Client Setup

- This section details how to configure and deploy the Velociraptor client on Windows, repacked with a custom configuration to connect to your Velociraptor server.

### Edit the Client Configuration

- On the Velociraptor server:

```sh
root@server:/opt/velociraptor# nano client.config.yaml
```

- Example configuration snippet:

```sh
Client:
  server_urls:
    - wss://<IP_SERVER>:8000/
```

- [Velociraptor Releases](https://github.com/Velocidex/velociraptor/releases)
- [Links Velociraptor](https://docs.velociraptor.app/downloads/)

### Download Velociraptor Binaries

```sh
root@server:/opt/velociraptor# wget https://github.com/Velocidex/velociraptor/releases/download/v0.73/velociraptor-v0.73.1-windows-amd64.exe
```

### Repack the Windows Client with Your Config

- Use the Linux Velociraptor binary to embed your configuration into the Windows client:

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.73.1-linux-amd64 config repack \
  --exe velociraptor-v0.73.1-windows-amd64.exe \
  client.config.yaml \
  windows-velociraptor-client.exe
```

### Host the Client Binary for Download

- Use Python’s simple HTTP server to make the repacked binary available to Windows clients:

```sh
root@server:/opt/velociraptor# python3 -m http.server 9999
```

- This serves files from the current directory at `http://<SERVER_IP>:9999/`.

![Velociraptor](/Threat-Hunting-Ubuntu/assets/13.png)

### Install the Client on Windows

- On the target Windows machine, download the binary and install it as a service:

```sh
PS C:\Users\Administrator\Downloads> .\windows-velociraptor-client.exe service install
```

- Once installed, the client will connect to the Velociraptor server and appear in the GUI:

![Velociraptor](/Threat-Hunting-Ubuntu/assets/14.png)

![Velociraptor](/Threat-Hunting-Ubuntu/assets/15.png)

### Verify Client Communication

- Run a simple Velociraptor query from the server to ensure the client responds:

```sh
SELECT *
FROM source(artifact="Windows.System.Pslist")
WHERE CommandLine != ""
```
