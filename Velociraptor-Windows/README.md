# Practical Threat Hunting

#### 1 - Velociraptor Server Set up

```sh
root@server:# apt update
root@server:# apt install wget openssl -y
root@server:# mkdir -p /opt/velociraptor
root@server:# cd /opt/velociraptor
```

- [Velociraptor](https://github.com/Velocidex/velociraptor/releases)
- [Links Velociraptor](https://docs.velociraptor.app/downloads/)

```sh
root@server:/opt/velociraptor# wget https://github.com/Velocidex/velociraptor/releases/download/v0.74/velociraptor-v0.74.1-linux-amd64

root@server:/opt/velociraptor# ls -al
root@server:/opt/velociraptor# chmod +x velociraptor-v0.74.1-linux-amd64

root@server:/opt/velociraptor# ./velociraptor-v0.74.1-linux-amd64 config generate -i
```

```sh
?
Welcome to the Velociraptor configuration generator
---------------------------------------------------

I will be creating a new deployment configuration for you. I will
begin by identifying what type of deployment you need.


What OS will the server be deployed on?
 linux
? Path to the datastore directory. /opt/velociraptor
?  Self Signed SSL
? What is the public DNS name of the Master Frontend (e.g. www.example.com): localhost
? Enter the frontend port to listen on. 8000
? Enter the port for the GUI to listen on. 8889
? Would you like to try the new experimental websocket comms?

Websocket is a bidirectional low latency communication protocol supported by
most modern proxies and load balancers. This method is more efficient and
portable than plain HTTP. Be sure to test this in your environment.
 Yes
? Would you like to use the registry to store the writeback files? (Experimental) No
? Which DynDns provider do you use? none
? GUI Username or email address to authorize (empty to end): admin
? GUI Username or email address to authorize (empty to end):
[INFO] 2025-01-21T17:54:21Z  _    __     __           _                  __
[INFO] 2025-01-21T17:54:21Z | |  / /__  / /___  _____(_)________ _____  / /_____  _____
[INFO] 2025-01-21T17:54:21Z | | / / _ \/ / __ \/ ___/ / ___/ __ `/ __ \/ __/ __ \/ ___/
[INFO] 2025-01-21T17:54:21Z | |/ /  __/ / /_/ / /__/ / /  / /_/ / /_/ / /_/ /_/ / /
[INFO] 2025-01-21T17:54:21Z |___/\___/_/\____/\___/_/_/   \__,_/ .___/\__/\____/_/
[INFO] 2025-01-21T17:54:21Z                                   /_/
[INFO] 2025-01-21T17:54:21Z Digging deeper!                  https://www.velocidex.com
[INFO] 2025-01-21T17:54:21Z This is Velociraptor 0.73.1 built on 2024-10-14T02:35:03Z (69c4fac)
[INFO] 2025-01-21T17:54:21Z Generating keys please wait....
? Path to the logs directory. /opt/velociraptor/logs
? Do you want to restrict VQL functionality on the server?

This is useful for a shared server where users are not fully trusted.
It removes potentially dangerous plugins like execve(), filesystem access etc.

NOTE: This is an experimental feature only useful in limited situations. If you
do not know you need it select N here!
 No
? Where should I write the server config file? server.config.yaml
? Where should I write the client config file? client.config.yaml
```

```sh
root@server:/opt/velociraptor# ls -l
total 59240
-rw------- 1 root root 2581 Jan 21 17:54 client.config.yaml
-rw------- 1 root root 12985 Jan 21 17:54 server.config.yaml
-rwxr-xr-x 1 root root 60641008 Oct 14 15:15 velociraptor-v0.74.1-linux-amd64
```

```sh
root@server:/opt/velociraptor# nano server.config.yaml
```

```sh
GUI:
  bind_address: <IP_SERVER>
  bind_port: 8889
```

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.74.1-linux-amd64 --config server.config.yaml debian server --binary velociraptor-v0.74.1-linux-amd64

root@server:/opt/velociraptor# dpkg -i velociraptor_server_0.73.1_amd64.deb

root@server:/opt/velociraptor# systemctl status velociraptor_server.service

root@server:/opt/velociraptor# ufw allow 8889/tcp
root@server:/opt/velociraptor# ufw allow 8000/tcp
```

```sh
https://<IP_SERVER>:8889
```

![Velociraptor](/Velociraptor-Linux/assets/01.png)

##### 2 - Velociraptor Client Set Up

```sh
root@server:/opt/velociraptor# nano client.config.yaml
```

```sh
Client:
  server_urls:
  - wss://<IP_SERVER>:8000/
```

```sh
root@server:/opt/velociraptor# wget https://github.com/Velocidex/velociraptor/releases/download/v0.74/velociraptor-v0.74.1-windows-amd64.exe
```

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.73.1-linux-amd64 config repack --exe velociraptor-v0.74.1-windows-amd64.exe client.config.yaml windows-velociraptor-client.exe
```

```sh
root@server:/opt/velociraptor# sha256sum windows-velociraptor-client.exe
root@server:/opt/velociraptor# md5sum windows-velociraptor-client.exe
```

```sh
root@server:/opt/velociraptor# ls -lh windows-velociraptor-client.exe
```

#### Téléchargement depuis un Client Windows

```sh
PS C:\Users\Administrator> Invoke-WebRequest -Uri "http://IP_SERVER_LINUX:9999/windows-velociraptor-client.exe" -OutFile "C:\Windows\Temp\velociraptor-client.exe"
PS C:\Users\Administrator> velociraptor-client.exe --version
PS C:\Users\Administrator> velociraptor-client.exe --config client.config.yaml config check
PS C:\Users\Administrator> velociraptor-client.exe service install
```

- Sécurité du Serveur Temporaire

```sh
root@server:/opt/velociraptor# python3 -m http.server 9999 --bind 127.0.0.1
```
