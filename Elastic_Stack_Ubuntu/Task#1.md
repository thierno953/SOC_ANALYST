# Investigation des tentatives d'accès SSH non autorisées avec ELK SIEM

- Vérifiez l'ELK et l'agent
- Simulez l'attaque et visualisez les événements
- Créez des règles de sécurité élastiques, détectez et enquêtez
- Réponse aux incidents

#### Vérification du bon fonctionnement d'ELK et de l'agent Fleet

- Accède à Kibana :

  - Vérifie que l’agent est online et collecte bien les logs système.

  `Management > Fleet > Fleet-Agent > View more agent metrics > Agent Info`

#### Simulation d’une attaque brute-force SSH et visualisation des logs

- Depuis la machine d’attaque (par exemple attack-ubuntu)

```sh
# Installer Hydra
sudo apt install hydra -y

# Créer une liste de mots de passe
echo -e "password\n123456\nadmin\nroot\nthierno" > password.txt

# Lancer une attaque brute-force SSH contre l’agent Fleet
hydra -l thierno -P password.txt <IP_FLEET_AGENT> ssh
```

#### Sur la machine Fleet Agent

```sh
# Observer les logs d’authentification en temps réel
tail -f /var/log/auth.log
```

- Répéter avec un autre utilisateur

```sh
hydra -l root -P password.txt <IP_FLEET_AGENT> ssh
```

```sh
tail -f /var/log/auth.log
```

#### Visualiser les événements dans Kibana Discover

- Aller dans: `Analytics > Discover`

- Filtrer avec KQL

```sh
event.outcome: "failure" and process.name: "sshd" and user.name: "thierno"
```

![ELK](/Elastic_Stack_Ubuntu/assets/03.png)

#### Création de règles de sécurité Elastic SIEM

- Aller dans :

  - `Security > Rules > Detection Rules > Add Elastic rules`

- Ajouter des règles existantes Elastic :

  - Cliquer sur Add Elastic rules

  - Rechercher ssh brute, installer et activer

  - Activer les règles pertinentes :

    - Potential Successful SSH Brute Force Attack

    - Potential External Linux SSH Brute Force

    - Potential Internal Linux SSH Brute Force

#### Génération et gestion d’une alerte de sécurité

- Déclencher à nouveau l'attaque pour générer une alerte

```sh
hydra -l thierno -P password.txt <IP_FLEET_AGENT> ssh
hydra -l root -P password.txt <IP_FLEET_AGENT> ssh

for i in {1..5}; do ssh fleet-server@<IP>; done
```

- Aller dans :

  - `Security > Alerts`

- Créer un cas d’investigation :

  - Cliquer sur l’alerte

  - `Take action > Add to new case`

  - Nom du cas : `SSH brute force attempt`

  - Description : `Tentative d'attaque par force brute SSH`

  - Valider avec `Create case`
