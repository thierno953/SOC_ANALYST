# HTTP

- HTTP est un protocole de la couche application du modèle OSI. Il repose sur le protocole TCP pour établir des connexions fiables entre le client et le serveur.

#### Fonctionnement

- Connexion TCP
  - Le client (navigateur) établit une connexion TCP avec le serveur via un processus en trois étapes (SYN, SYN-ACK, ACK).
- Requête HTTP
  - Le client envoie une requête HTTP contenant
    - Une méthode (comme GET, POST).
    - Une URL (par exemple, /index.html).
    - Des en-têtes (comme Host, User-Agent).
- Réponse HTTP
  - Le serveur renvoie une réponse contenant
    - Un code d'état (exemple : 200 OK ou 404 Not Found).
    - Les en-têtes de réponse.
    - Le contenu demandé (comme une page HTML ou une image).
- Fermeture de la connexion :

  - La connexion TCP peut être fermée après l'échange ou réutilisée pour d'autres requêtes (HTTP/1.1 et versions ultérieures).

- HTTP est un protocole essentiel pour le fonctionnement du Web. Il utilise un modèle client-serveur basé sur des requêtes et réponses pour permettre l'échange de ressources sur Internet.
