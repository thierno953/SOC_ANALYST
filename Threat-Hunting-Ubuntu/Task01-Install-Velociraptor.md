# Threat Hunting with Velociraptor

## Velociraptor Server Setup

### Update and Install Prerequisites

```sh
root@server:~# apt update
root@server:~# apt install wget openssl -y
root@server:~# mkdir -p /opt/velociraptor
root@server:~# cd /opt/velociraptor
```

### References

- [Velociraptor GitHub Releases](https://github.com/Velocidex/velociraptor/releases)
- [Velociraptor Documentation](https://docs.velociraptor.app/downloads/)

### Download the Velociraptor Binary

```sh
root@server:/opt/velociraptor# wget https://github.com/Velocidex/velociraptor/releases/download/v0.73/velociraptor-v0.73.1-linux-amd64
root@server:/opt/velociraptor# chmod +x velociraptor-v0.73.1-linux-amd64
```

### Generate Server Configuration

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.73.1-linux-amd64 config generate -i
root@server:/opt/velociraptor# nano server.config.yaml
```

#### Edit the configuration to bind to the server IP:

```sh
GUI:
  bind_address: <SERVER_IP>
  bind_port: 8889
```

### Build and Install the Debian Server Package

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.73.1-linux-amd64 --config server.config.yaml debian server --binary velociraptor-v0.73.1-linux-amd64
root@server:/opt/velociraptor# dpkg -i velociraptor_server_0.73.1_amd64.deb
```

### Enable Firewall Rules

```sh
root@server:/opt/velociraptor# ufw allow 8889/tcp
root@server:/opt/velociraptor# ufw allow 8000/tcp
```

### Verify Server Status

```sh
root@server:/opt/velociraptor# systemctl status velociraptor_server.service
```

#### The Velociraptor GUI is now accessible at:

```sh
https://<SERVER_IP>:8889
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/01.png)

## Velociraptor Client Setup

### Generate Client Configuration

```sh
root@server:/opt/velociraptor# nano client.config.yaml
```

```sh
Client:
  server_urls:
    - wss://<SERVER_IP>:8000/
```

### Build Debian Client Package

```sh
root@server:/opt/velociraptor# ./velociraptor-v0.73.1-linux-amd64 --config client.config.yaml debian client
```

### Transfer Package to the Client

```sh
root@server:/opt/velociraptor# scp velociraptor_client_0.73.1_amd64.deb agent01@<AGENT_IP>:/tmp
```

### Install Client on Linux Endpoint

```sh
root@agent01:/tmp# dpkg -i velociraptor_client_0.73.1_amd64.deb
root@agent01:/tmp# systemctl status velociraptor_client.service
```

![Velociraptor](/Threat-Hunting-Ubuntu/assets/02.png)

![Velociraptor](/Threat-Hunting-Ubuntu/assets/03.png)
