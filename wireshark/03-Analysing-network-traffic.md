# Analyse du trafic réseau

#### Trouver les meilleurs locuteurs (hôtes avec le plus de trafic)

- Tâche 1 : Trouver les meilleurs locuteurs (hôte avec le plus de trafic).
- **Objectif**: Détecter les retransmissions TCP et les problèmes de connexion.

![Wireshark](/assets/15.png)
![Wireshark](/assets/16.png)

#### Détecter les problèmes de connexion

- Tâche 2 : identifier le problème de connexion.
- **Objectif**: Détecter les retransmissions TCP et les problèmes de connexion.

```sh
1. Le client envoie un paquet TCP au serveur
   Client --------------------> Serveur
          (Paquet envoyé)

2. Le serveur doit répondre avec un ACK (acquittement)
   Client <-------------------- Serveur
          (ACK reçu)

3. Si le client n'obtient pas l'ACK à temps (timeout), il retransmet le même paquet
   Client --------------------> Serveur
          (Paquet retransmis)

4. Le serveur finit par répondre avec un ACK
   Client <-------------------- Serveur
          (ACK reçu)
```

![Wireshark](/assets/17.png)
![Wireshark](/assets/18.png)
![Wireshark](/assets/19.png)
![Wireshark](/assets/20.png)

- Tâche 3: Analyse du trafic ICMP (Ping).
- **Objectif**: Examiner les requêtes et les réponses ICMP.

```sh
|----------------------|
|      Request         |
| A ----------------> B|

# ping microsoft.com -4
# ping microsoft.com -t4
```
