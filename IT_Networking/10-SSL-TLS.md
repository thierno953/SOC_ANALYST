# SSL & TLS

- **SSL (Secure Sockets Layer)** : Protocole utilisé pour établir des connexions chiffrées entre un serveur web et un navigateur. Il garantit que toutes les données échangées restent privées et intègres.
- **TLS (Transport Layer Security)** : Version améliorée et plus sécurisée de SSL. TLS offre des méthodes de chiffrement plus robustes et une meilleure sécurité.
- **Versions de TLS** : TLS 1.0, 1.1, 1.2, et 1.3 (la dernière version, la plus rapide et sécurisée)

#### SSL/TLS Handshake

- Le processus d'établissement d'une connexion sécurisée entre le client (navigateur) et le serveur se déroule en plusieurs étapes :
  - Le client envoie une requête au serveur pour établir une connexion sécurisée.
  - Le serveur répond avec son certificat SSL/TLS pour prouver son identité.
  - Les clés de chiffrement sont échangées à l'aide de cryptographie asymétrique pour établir une clé symétrique partagée.
  - Une fois la clé partagée établie, toutes les communications sont chiffrées avec cette clé symétrique.

#### Utilisations de SSL/TLS

- **Navigation web** : Sécurise les sites web via HTTPS.
- **Communication par e-mail** : Protège les échanges via SMTP, IMAP ou POP3.
- **Connexions VPN** : Chiffre les données dans les tunnels VPN.
- **Applications de messagerie instantanée** : Sécurise les messages envoyés en temps réel.
- **Protocoles de transfert de fichiers** : Protège les transferts via FTPS ou SFTP.
- **APIs** : Sécurise les communications entre clients et serveurs pour protéger les données échangées.

- SSL/TLS est indispensable pour sécuriser les communications sur Internet en garantissant confidentialité, intégrité et authenticité des données échangées. TLS est désormais la norme moderne, remplaçant SSL qui est obsolète.
