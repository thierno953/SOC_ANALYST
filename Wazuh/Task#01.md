# Wazuh Installation & Integration 

## PREPARE THE SYSTEM

#### Switch to root

```sh
sudo su
```

#### Extend root logical volume (if needed)

```sh
df -h /
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
df -h /
```

## DOWNLOAD INSTALLATION FILES

- Reference: [https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html](https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html)

```sh
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.12/config.yml
```

#### Edit configuration

```sh
nano config.yml
```

```sh
nodes:
  indexer:
    - name: node-1
      ip: "<indexer-node-ip>"

  server:
    - name: wazuh-1
      ip: "<wazuh-manager-ip>"

  dashboard:
    - name: dashboard
      ip: "<dashboard-node-ip>"
```

#### Generate config files

```sh
bash wazuh-install.sh --generate-config-files
```

## INSTALL WAZUH INDEXER

```sh
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
bash wazuh-install.sh --wazuh-indexer node-1
```

#### Start the indexer cluster

```sh
bash wazuh-install.sh --start-cluster
```

#### Get admin password from generated files

```sh
tar -axf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt -O | grep -P "\'admin\'" -A 1
```

#### Test indexer access (replace <ADMIN_PASSWORD> and <WAZUH_INDEXER_IP>)

```sh
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200
curl -k -u admin:<ADMIN_PASSWORD> https://<WAZUH_INDEXER_IP>:9200/_cat/nodes?v
```

## INSTALL WAZUH SERVER

- Reference: [https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html](https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html)

```sh
bash wazuh-install.sh --wazuh-server wazuh-1
```

## INSTALL WAZUH DASHBOARD

- Reference: [https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/installation-assistant.html](https://documentation.wazuh.com/current/installation-guide/wazuh-dashboard/installation-assistant.html)

```sh
bash wazuh-install.sh --wazuh-dashboard dashboard
```

## CONFIGURE FIREWALL

```sh
sudo ufw allow 1514/tcp   # Wazuh agent port
sudo ufw allow 55000/tcp  # Wazuh API
sudo ufw allow 9200/tcp   # Wazuh indexer
sudo ufw allow 5601/tcp   # Wazuh dashboard
sudo ufw allow OpenSSH    # SSH access
sudo ufw enable
```

## CONFIGURE VULNERABILITY DETECTION

#### Enable vulnerability detection

```sh
nano /var/ossec/etc/internal_options.conf
```

```sh
# Change the following line:
# From:
# vulnerability-detection.disable_scan_manager=1
# To:
# vulnerability-detection.disable_scan_manager=0
```

## CONFIGURE LOG MONITORING

```sh
nano /var/ossec/etc/ossec.conf
```

#### Add the following inside <ossec_config>:

```sh
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/wazuh/wazuh.log</location>
</localfile>
```

## RESTART WAZUH MANAGER

```sh
sudo systemctl restart wazuh-manager
```
