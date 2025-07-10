# SFTP vs FTPS

#### SFTP (SSH File Transfert Protocol)

- SFTP est un protocole qui offre des capacités de transfert de fichiers sécurisés via une connexion SSH.
- **Authentication**: Utilise des mécanismes basés sur SSH, prenant en charge l'authentification par clé ou mot de passe.
- **Port**: Fonctionne sur le port 22
- **Use Cases**: Idéal pour les transferts sécurisés dans des environnements où SSH est déjà utilisé ou requis.

#### FTPS (FTP Secure)

- FTPS est une extension du protocole FTP qui ajoute la prise en charge du chiffrement via TLS (Transport Layer Security).
- **Security**: Chiffre les canaux de données et de contrôle, offrant une protection pendant les transferts de fichiers.
- **Authentication**: Prend en charge l'authentification par certificat et les combinaisons nom d'utilisateur/mot de passe.
- **Use Cases**: Convient aux environnements nécessitant une conformité à des normes comme PCI-DSS, où les transferts chiffrés sont essentiels.
