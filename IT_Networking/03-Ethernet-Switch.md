# Ethernet Switch

- Un switch Ethernet fonctionne à la couche 2 (liaison de données) du modèle OSI et utilise une table d'adresses MAC pour acheminer les trames de données.

#### Fonctionnement

- Le switch apprend dynamiquement les adresses MAC des appareils connectés à ses ports.

- Lorsqu'il reçoit une trame, il vérifie l'adresse MAC de destination

  - Si elle est connue dans sa table, il envoie la trame sur le port correspondant.
  - Si elle est inconnue, il diffuse la trame (broadcast) sur tous les ports sauf celui d'origine.

- Chaque port du switch constitue un domaine de collision indépendant, ce qui améliore les performances par rapport à un hub.

- Un switch Ethernet optimise la communication dans un réseau local en dirigeant efficacement le trafic entre les appareils connectés
