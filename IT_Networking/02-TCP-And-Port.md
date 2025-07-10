# TCP et des ports avec une analogie

- TCP est un protocole de la couche transport (OSI Layer 4) qui établit une connexion fiable entre deux appareils. Il utilise des ports pour identifier les applications ou services spécifiques.
- Chaque paquet envoyé contient :

  - Une adresse IP source et destination (pour identifier les appareils).
  - Un port source et destination (pour identifier les applications/services).

#### Fonctionnement des Ports

- Les ports sont numérotés de 1 à 65535.

  - Ports bien connus (1-1023) : Réservés aux services standards (ex. HTTP = 80, HTTPS = 443).
  - Ports éphémères (1024-65535) : Utilisés temporairement par les clients pour initier une connexion.

- TCP garantit une communication fiable entre appareils, tandis que les ports permettent d'identifier quel service ou application doit recevoir les données.
