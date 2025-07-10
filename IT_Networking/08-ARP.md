# ARP(Address Resolution Protocol)

- L'ARP fonctionne à l'interface entre la couche 2 (liaison de données) et la couche 3 (réseau) du modèle OSI. Il est utilisé pour résoudre les adresses IPv4 en adresses MAC dans les réseaux Ethernet.

#### Fonctionnement d'ARP

- Lorsqu'un appareil doit envoyer une trame Ethernet à une destination sur le même sous-réseau, il vérifie d'abord son cache ARP pour voir si l'adresse MAC correspondante est déjà connue.
- Si l'adresse MAC n'est pas dans le cache, il envoie une requête ARP broadcast sur le réseau local :
  - Exemple : "Qui a l'adresse IP 192.168.1.5 ? Répondez avec votre adresse MAC."
- Tous les appareils reçoivent la requête, mais seul celui qui possède cette adresse IP répond avec un message ARP contenant son adresse MAC.
- L'appareil émetteur met à jour son cache ARP avec cette correspondance IP-MAC pour de futures communications.

- l'ARP est essentiel dans les réseaux IPv4 pour traduire les adresses logiques (IP) en adresses physiques (MAC), permettant ainsi la communication au niveau de la couche liaison dans un réseau local.
