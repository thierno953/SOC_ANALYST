# Integrating Wazuh with Shuffle

- Shuffle Installation: [https://github.com/shuffle/shuffle/blob/main/.github/install-guide.md](https://github.com/shuffle/shuffle/blob/main/.github/install-guide.md)
- Based on the official guide: [https://wazuh.com/blog/integrating-wazuh-with-shuffle/](https://wazuh.com/blog/integrating-wazuh-with-shuffle/)

#### Step-by-Step Integration Instructions

- Open Wazuh Manager Configuration

```sh
sudo nano /var/ossec/etc/ossec.conf
```

#### Add the Shuffle Integration Block

- Insert the following XML block inside the `<ossec_config>` section:

```sh
<integration>
  <name>shuffle</name>
  <hook_url>https://<YOUR_SHUFFLE_URL>/api/v1/hooks/<HOOK_ID></hook_url>
  <level>3</level>
  <alert_format>json</alert_format>
</integration>
```

#### Restart the Wazuh Manager

```sh
sudo systemctl restart wazuh-manager
```


