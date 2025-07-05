#### Surveillance d'intégrité des fichiers (FIM)

- [Documentation officielle](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/how-to-configure-fim.html)

- Crée un répertoire à surveiller

```sh
cd /home/agent01
mkdir documents && cd documents
```

- Modifie la configuration de Wazuh Agent

```sh
sudo nano /var/ossec/etc/ossec.conf
```

- Ajoute ou modifie cette section :

```sh
<syscheck>
    <directories check_all="yes" report_changes="yes" realtime="yes">/home/agent01/documents</directories>
</syscheck>
```

- Redémarre l'agent Wazuh

```sh
sudo systemctl restart wazuh-agent
```

- Crée ou supprime des fichiers pour tester la détection

```sh
touch prueba.txt
rm prueba.txt

mkdir /home/agent01/documents/prueba_dir
echo "test" > /home/agent01/documents/prueba_dir/test.txt
```

- Sur le Wazuh Manager (interface web), tu devrais voir ces événements dans l’onglet **SIEM**.

## Détection de malwares avec YARA

#### Documentation officielle YARA + Wazuh

- [Documentation officielle YARA + Wazuh](https://documentation.wazuh.com/current/user-manual/capabilities/malware-detection/fim-yara.html)

#### Installer YARA

```sh
sudo apt update
sudo apt install -y make gcc autoconf libtool libssl-dev pkg-config
sudo curl -LO https://github.com/VirusTotal/yara/archive/v4.2.3.tar.gz
sudo tar -xvzf v4.2.3.tar.gz -C /usr/local/bin/ && rm -f v4.2.3.tar.gz
cd /usr/local/bin/yara-4.2.3/
sudo ./bootstrap.sh && sudo ./configure && sudo make && sudo make install && sudo make check
```

#### Configure les bibliothèques :

```sh
sudo su
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
exit
```

#### Télécharger des règles YARA (exemple Valhalla)

```sh
sudo mkdir -p /tmp/yara/rules
sudo curl 'https://valhalla.nextron-systems.com/api/v1/get' \
-H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8' \
-H 'Accept-Language: en-US,en;q=0.5' \
--compressed \
-H 'Referer: https://valhalla.nextron-systems.com/' \
-H 'Content-Type: application/x-www-form-urlencoded' \
-H 'DNT: 1' -H 'Connection: keep-alive' -H 'Upgrade-Insecure-Requests: 1' \
--data 'demo=demo&apikey=1111111111111111111111111111111111111111111111111111111111111111&format=text' \
-o /tmp/yara/rules/yara_rules.yar
```

#### Créer un script `yara.sh` pour la réponse active

```sh
sudo nano /var/ossec/active-response/bin/yara.sh
```

```sh
#!/bin/bash
# Wazuh - Yara active response
# Copyright (C) 2015-2022, Wazuh Inc.
#
# This program is free software; you can redistribute it
# and/or modify it under the terms of the GNU General Public
# License (version 2) as published by the FSF - Free Software
# Foundation.


#------------------------- Gather parameters -------------------------#

# Extra arguments
read INPUT_JSON
YARA_PATH=$(echo $INPUT_JSON | jq -r .parameters.extra_args[1])
YARA_RULES=$(echo $INPUT_JSON | jq -r .parameters.extra_args[3])
FILENAME=$(echo $INPUT_JSON | jq -r .parameters.alert.syscheck.path)

# Set LOG_FILE path
LOG_FILE="logs/active-responses.log"

size=0
actual_size=$(stat -c %s ${FILENAME})
while [ ${size} -ne ${actual_size} ]; do
    sleep 1
    size=${actual_size}
    actual_size=$(stat -c %s ${FILENAME})
done

#----------------------- Analyze parameters -----------------------#

if [[ ! $YARA_PATH ]] || [[ ! $YARA_RULES ]]
then
    echo "wazuh-yara: ERROR - Yara active response error. Yara path and rules parameters are mandatory." >> ${LOG_FILE}
    exit 1
fi

#------------------------- Main workflow --------------------------#

# Execute Yara scan on the specified filename
yara_output="$("${YARA_PATH}"/yara -w -r "$YARA_RULES" "$FILENAME")"

if [[ $yara_output != "" ]]
then
    # Iterate every detected rule and append it to the LOG_FILE
    while read -r line; do
        echo "wazuh-yara: INFO - Scan result: $line" >> ${LOG_FILE}
    done <<< "$yara_output"
fi

exit 0;
```

#### Donne les permissions :

```sh
sudo chmod 750 /var/ossec/active-response/bin/yara.sh
sudo chown root:wazuh /var/ossec/active-response/bin/yara.sh
```

#### Installer jq :

```sh
sudo apt install -y jq
```

#### Configuration YARA dans Wazuh

- `ossec.conf` (encore une fois)

```sh
sudo nano /var/ossec/etc/ossec.conf
```

```sh
<syscheck>
  <directories realtime="yes">/root/</directories>
</syscheck>
```

```sh
sudo systemctl restart wazuh-agent
```

## Ajout des décoders et règles

#### Fichier `local_decoder.xml`

```sh
root@wazuh:/# nano /var/ossec/etc/decoders/local_decoder.xml
```

#### Ajoute :

```sh
<decoder name="yara_decoder">
  <prematch>wazuh-yara:</prematch>
</decoder>

<decoder name="yara_decoder1">
  <parent>yara_decoder</parent>
  <regex>wazuh-yara: (\S+) - Scan result: (\S+) (\S+)</regex>
  <order>log_type, yara_rule, yara_scanned_file</order>
</decoder>
```

#### Fichier `local_rules.xml`

```sh
root@wazuh:/# nano /var/ossec/etc/rules/local_rules.xml
```

#### Ajoute :

```sh
<group name="syscheck,">
  <rule id="100200" level="7">
    <if_sid>550</if_sid>
    <field name="file">/root/</field>
    <description>File modified in /root directory.</description>
  </rule>
  <rule id="100201" level="7">
    <if_sid>554</if_sid>
    <field name="file">/root/</field>
    <description>File added to /root directory.</description>
  </rule>
</group>

<group name="yara,">
  <rule id="108000" level="0">
    <decoded_as>yara_decoder</decoded_as>
    <description>Yara grouping rule</description>
  </rule>
  <rule id="108001" level="12">
    <if_sid>108000</if_sid>
    <match>wazuh-yara: INFO - Scan result: </match>
    <description>File "$(yara_scanned_file)" is a positive match. Yara rule: $(yara_rule)</description>
  </rule>
</group>
```

#### Dans le serveur Wazuh

```sh
root@wazuh:/# nano /var/ossec/etc/ossec.conf
```

#### Ajoute :

```sh
<ossec_config>
  <command>
    <name>yara_linux</name>
    <executable>yara.sh</executable>
    <extra_args>-yara_path /usr/local/bin -yara_rules /tmp/yara/rules/yara_rules.yar</extra_args>
    <timeout_allowed>no</timeout_allowed>
  </command>

  <active-response>
    <disabled>no</disabled>
    <command>yara_linux</command>
    <location>local</location>
    <rules_id>100200,100201</rules_id>
  </active-response>
</ossec_config>
```

#### Redémarre le manager

```sh
root@wazuh:/# service wazuh-manager restart
```

## Tester avec des fichiers malware

#### Télécharger des échantillons

```sh
root@agent01:/usr/local/bin/yara-4.2.3# curl https://wazuh-demo.s3-us-west-1.amazonaws.com/mirai --output ~/mirai
curl https://wazuh-demo.s3-us-west-1.amazonaws.com/xbash --output ~/Xbash
sudo mv ~/mirai /root/
sudo mv ~/Xbash /root/
```

#### Télécharger les logiciels malveillants de test.

```sh
root@agent01:/home/agent01/documents# nano malware.sh
```

#### COPIER

```sh
#!/bin/bash
# Wazuh - Malware Downloader for test purposes

function fetch_sample(){
  curl -s -XGET "$1" -o "$2"
}

echo "WARNING: Downloading Malware samples, please use this script with caution."
read -p "  Do you want to continue? (y/n) " -n 1 -r ANSWER
echo

if [[ $ANSWER =~ ^[Yy]$ ]]; then
    echo

    mkdir -p /root/malware_samples

    # Mirai
    echo "# Mirai: https://en.wikipedia.org/wiki/Mirai_(malware)"
    echo "Downloading malware sample..."
    fetch_sample "https://wazuh-demo.s3-us-west-1.amazonaws.com/mirai" "/root/malware_samples/mirai" && echo "Done!" || echo "Error while downloading."
    echo

    # Xbash
    echo "# Xbash: https://unit42.paloaltonetworks.com/unit42-xbash-combines-botnet-ransomware-coinmining-worm-targets-linux-windows/"
    echo "Downloading malware sample..."
    fetch_sample "https://wazuh-demo.s3-us-west-1.amazonaws.com/xbash" "/root/malware_samples/xbash" && echo "Done!" || echo "Error while downloading."
    echo

    # VPNFilter
    echo "# VPNFilter: https://news.sophos.com/en-us/2018/05/24/vpnfilter-botnet-a-sophoslabs-analysis/"
    echo "Downloading malware sample..."
    fetch_sample "https://wazuh-demo.s3-us-west-1.amazonaws.com/vpn_filter" "/root/malware_samples/vpn_filter" && echo "Done!" || echo "Error while downloading."
    echo

    # Webshell
    echo "# WebShell: https://github.com/SecWiki/WebShell-2/blob/master/Php/Worse%20Linux%20Shell.php"
    echo "Downloading malware sample..."
    fetch_sample "https://wazuh-demo.s3-us-west-1.amazonaws.com/webshell" "/root/malware_samples/webshell" && echo "Done!" || echo "Error while downloading."
    echo
fi
```

#### Exécuter le script

```sh
root@agent01:/home/agent01/documents# bash malware.sh
```
