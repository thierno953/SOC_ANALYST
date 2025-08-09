# Velociraptor Installation and Agent Deployment

#### Install Velociraptor on Linux Server

```sh
sudo apt update -y
```

#### Download Velociraptor (check for latest version if needed)

- Reference: [https://github.com/Velocidex/velociraptor/releases](https://github.com/Velocidex/velociraptor/releases)

```sh
mkdir velo && cd velo
wget https://github.com/Velocidex/velociraptor/releases/download/v0.72.0/velociraptor-v0.72.0-linux-amd64

# Move the binary to a system path
sudo cp velociraptor-v0.72.0-linux-amd64 /usr/local/bin/velociraptor
sudo chmod +x /usr/local/bin/velociraptor
```

#### Generate Configuration

```sh
velociraptor config generate -i
sudo mv *.config.yaml /etc/
```

#### Modify Server Configuration

```sh
sudo nano /etc/server.config.yaml
```

- Edit the `bind_address` line to reflect the VM’s IP address:

```sh
bind_address: <VM-IP>
```

#### Create Systemd Service for Velociraptor

```sh
sudo nano /lib/systemd/system/velociraptor.service
```

- Paste:

```sh
[Unit]
Description=Velociraptor linux amd64
After=syslog.target network.target

[Service]
Type=simple
Restart=always
RestartSec=120
LimitNOFILE=20000
Environment=LANG=en_US.UTF-8
ExecStart=/usr/local/bin/velociraptor --config /etc/server.config.yaml frontend -v

[Install]
WantedBy=multi-user.target
```

#### Start Velociraptor Service

```sh
sudo systemctl daemon-reload
sudo systemctl enable --now velociraptor
systemctl status velociraptor
```

- Access the UI: `https://<VM-IP>:8889/`

## Deploying Velociraptor Agent on Windows

#### Download the Windows MSI

```sh
wget https://github.com/Velocidex/velociraptor/releases/download/v0.72.0/velociraptor-v0.72.0-windows-amd64.msi
```

#### Repack the MSI with Client Config

```sh
# Ensure the Linux binary is executable (must be the same version)
chmod +x /usr/local/bin/velociraptor

/usr/local/bin/velociraptor config repack \
  --msi velociraptor-v0.72.0-windows-amd64.msi \
  /etc/client.config.yaml \
  windows-agent.msi
```

#### Install the Agent on Windows

- Double-click `windows-agent.msi` and run it as Administrator

#### Deploying Velociraptor Agent on Linux

- Build the Linux Agent (`.deb`) from Config

```sh
velociraptor --config /etc/client.config.yaml debian client
```

#### Install the .deb Agent Package

```sh
sudo dpkg -i velociraptor_client_0.72.0_amd64.deb
```

#### Memory Acquisition

```sh
velociraptor --config /etc/client.config.yaml \
  artifacts collect Windows.Memory.Acquisition \
  --output memdump.mem
```

#### Immediate Analysis with Volatility

```sh
python3 vol.py -f memdump.mem windows.pslist.PsList
python3 vol.py -f memdump.mem windows.netscan.NetScan
python3 vol.py -f memdump.mem windows.malfind
```
