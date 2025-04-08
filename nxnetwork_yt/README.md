```sh
# Installer BIND9
sudo apt update && sudo apt install -y bind9 bind9utils

# Créer le fichier de zone
sudo nano /etc/bind/db.nextcloud.lan
```

```sh
; Zone DNS pour nextcloud.lan
$TTL    86400
@       IN      SOA     ns1.nextcloud.lan. admin.nextcloud.lan. (
                        2023080101 ; Serial
                        3600       ; Refresh
                        1800       ; Retry
                        604800     ; Expire
                        86400      ; Minimum TTL
)

; Enregistrements
@       IN      NS      ns1.nextcloud.lan.
@       IN      A       192.168.129.52

ns1     IN      A       192.168.129.52
cloud   IN      A       192.168.129.52
office  IN      A       192.168.129.52
talk    IN      A       192.168.129.52
```

```sh
# Activer la zone
sudo nano /etc/bind/named.conf.local
```

```sh
zone "nextcloud.lan" {
    type master;
    file "/etc/bind/db.nextcloud.lan";
    allow-transfer { none; };
};
```

```sh
# Vérifier et redémarrer
sudo named-checkzone nextcloud.lan /etc/bind/db.nextcloud.lan
sudo systemctl restart bind9
```

```sh
nano docker-compose.yml
```

```sh
version: '3'

services:
  nextcloud:
    image: nextcloud:latest
    hostname: cloud.nextcloud.lan
    networks:
      - nextcloud_net
    ports:
      - "80:80"
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - NEXTCLOUD_TRUSTED_DOMAINS=cloud.nextcloud.lan
      - OVERWRITEHOST=cloud.nextcloud.lan
      - OVERWRITEPROTOCOL=http
      - MYSQL_HOST=db.nextcloud.lan
      - MYSQL_PASSWORD=nextcloud123
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - REDIS_HOST=redis.nextcloud.lan

  db:
    image: mariadb:10.6
    hostname: db.nextcloud.lan
    networks:
      - nextcloud_net
    environment:
      - MYSQL_ROOT_PASSWORD=root123
      - MYSQL_PASSWORD=nextcloud123
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    volumes:
      - db_data:/var/lib/mysql

  redis:
    image: redis:alpine
    hostname: redis.nextcloud.lan
    networks:
      - nextcloud_net

  collabora:
    image: collabora/code:latest
    hostname: office.nextcloud.lan
    networks:
      - nextcloud_net
    ports:
      - "9980:9980"
    environment:
      - domain=cloud\\.nextcloud\\.lan
    restart: unless-stopped

  talk:
    image: nextcloud/spreed-signaling:latest
    hostname: talk.nextcloud.lan
    networks:
      - nextcloud_net
    ports:
      - "8081:8081"
    depends_on:
      - redis

networks:
  nextcloud_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/24

volumes:
  nextcloud_data:
  db_data:
```

```sh
# Configurer le client DNS (sur le serveur et les machines clientes)
sudo nano /etc/resolv.conf
```
```sh
nameserver 192.168.129.52
search nextcloud.lan
options edns0 trust-ad
```

```sh
# Lancer les containers
docker-compose up -d

# Vérifier le DNS
nslookup cloud.nextcloud.lan 192.168.129.52

# Accéder à Nextcloud
firefox http://cloud.nextcloud.lan
```

```sh
⚙️ 5. Configuration des Services
#Nextcloud :

Complétez l'installation via l'interface web

Base de données : db.nextcloud.lan / utilisateur nextcloud

#Collabora :

Apps → "Collabora Online" → URL : http://office.nextcloud.lan:9980

Talk :

Apps → "Talk" → Signaling Server : ws://talk.nextcloud.lan:8081

📜 Fichiers de Configuration Clés
#BIND9 :

/etc/bind/db.nextcloud.lan : Zone DNS

/etc/bind/named.conf.local : Déclaration de zone

#Docker :

docker-compose.yml : Tout-en-un réseau isolé

#Résolution DNS :

/etc/resolv.conf : Sur toutes les machines du réseau
```

```sh
# Vérifier les logs BIND9
sudo journalctl -u bind9 -f

# Tester la résolution
dig @192.168.129.52 cloud.nextcloud.lan

# Inspecter le réseau Docker
docker network inspect nextcloud_nextcloud_net
```

```sh
# 1. Mise à jour du système
sudo apt update && sudo apt upgrade -y

# 2. Pare-feu (UFW)
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw allow 22/tcp       # SSH
sudo ufw allow 53/tcp       # DNS (BIND9)
sudo ufw allow 53/udp
sudo ufw allow 80/tcp       # Nextcloud
sudo ufw allow 443/tcp      # (Pour HTTPS futur)
sudo ufw enable

# 3. Désactiver les services inutiles
sudo systemctl stop apache2 && sudo systemctl disable apache2
```

```sh
# 1. Chroot BIND9 (isolation)
sudo nano /etc/default/bind9
```

```sh
# 2. Restreindre les requêtes DNS
sudo nano /etc/bind/named.conf.options
```

```sh
options {
    allow-query { 192.168.1.0/24; };  # Votre LAN seulement
    recursion no;                      # Désactive la récursion
    dnssec-validation yes;             # Validation DNSSEC
};
```

```sh
# 1. Politique de conteneurs
echo '{"userns-remap": "default"}' | sudo tee /etc/docker/daemon.json
sudo systemctl restart docker

# 2. Limites de ressources (dans docker-compose.yml)
services:
  nextcloud:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
    security_opt:
      - no-new-privileges:true
    read_only: true  # Sauf pour les volumes
```

```sh
# 1. Configuration de base sécurisée
docker exec -it nextcloud_app occ config:system:set \
  --type=string \
  --value="0.0.0.0/0" \
  trusted_proxies
```

```sh
# 2. Fichier config.php renforcé
sudo nano /var/lib/docker/volumes/nextcloud_nextcloud_data/_data/config/config.php
```

```sh
  'passwordsalt' => 'votre_salt_unique_ici',
  'secret' => 'votre_secret_ici',
  'trusted_domains' =>
  array (
    0 => 'cloud.nextcloud.lan',
  ),
  'forceSSL' => false,  // À passer en true si vous ajoutez HTTPS
  'auth.bruteforce.protection.enabled' => true,
  'filelocking.enabled' => true,
  'allow_local_remote_servers' => false,
```

```sh
# 1. Surveillance des logs
sudo apt install fail2ban
```

```sh
sudo nano /etc/fail2ban/jail.d/nextcloud.conf
```

```sh
[nextcloud]
enabled = true
port = 80,443
filter = nextcloud
logpath = /var/lib/docker/volumes/nextcloud_nextcloud_data/_data/data/nextcloud.log
maxretry = 3
```

```sh
# 2. Mises à jour automatiques
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades  # Choisir "Yes"
```

```sh
# 1. Générer un certificat auto-signé
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/nextcloud.key \
  -out /etc/ssl/certs/nextcloud.crt

# 2. Configurer Nginx en reverse proxy HTTPS
# (Voir le guide précédent adapté pour .lan)
```
