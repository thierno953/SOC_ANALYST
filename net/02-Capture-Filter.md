# Wireshark

## Présentation de Wireshark

#### Parcourez Wireshark

#### Analyse du trafic réseau

#### Enquête de sécurité

#### Qu'est-ce que Wireshark ?

- Wireshark est un analyseur de protocole réseau qui capture et inspecte les données circulant sur un réseau en temps réel.

- Il capture les paquets (petits morceaux de données) lorsqu'ils sont envoyés sur un réseau et les décode dans un format lisible par l'homme.

- CAS D'UTILISATION COURANTS:
  - Dépannage réseau.
  - Surveillance de sécurité et criminalistique.
  - Apprentissage et analyse du comportement du réseau.

# Principales caractéristiques de Wireshark

```sh
---------------------------------- |-------------------------------|
Capture de paquets en direct       | Filtrage et recherche         |
-----------------------------------|-------------------------------|
Inspection approfondie des paquets | Enregistrement et exportation |
-----------------------------------|-------------------------------|
```

## Applications pratiques de Wireshark

- Dépannage réseau.
- Enquêtes de sécurité
- Analyse et apprentissage des protocoles

## Découvrez Wireshark

#### Filtre de capture vs. Filtre d'affichage

- **Filtre de capture:** appliqué avant le début du processus de capture de paquets, pour limiter les paquets capturés.

- **Filtre d'affichage:** appliqué après la capture des paquets, pour filtrer ce qui est affiché dans l'outil d'analyse.

```sh
--------------------------|-------------------------|---------------------------|
Fonctionnalité            | Filtres de capture      | Filtres d'affichage     --|
--------------------------|-------------------------|---------------------------|
Filtre d'adresse IP       | host 192.168.129.1      | ip.src=192.168.129.1    --|
--------------------------|-------------------------|---------------------------|
Trafic TCP                | tcp                     | tcp                     --|
--------------------------|-------------------------|---------------------------|
Trafic HTTP               | tcp.port 80             | http.request            --|
--------------------------|-------------------------|---------------------------|
Trafic de sous-réseau     | net 192.168.129.0/24    | ip.addr=192.168.129.0/24  |
--------------------------|-------------------------|---------------------------|
Paquets entre hosts       | host 192.168.129.1      | ip.src=192.168.129.1      |
                          |         et              |        et                 |
                          | host 192.168.129.2      | ip.dst=192.168.129.2      |
--------------------------|-------------------------|---------------------------|
DNS                       | port udp 53             | dns                       |
--------------------------|-------------------------|---------------------------|
```

## Enquête sur le réseau

#### Filtre d'affichage et filtre de capture

![Wireshark](/assets/0.png)
![Wireshark](/assets/00.png)

#### Profils Wireshark

##### Interface principale où vous travaillez, c'est ce q'uon appelle pan pan one, pan pan two et pan pan three.

- 1 - Le premier vous montre tout le traffic, ce qui signifie à qu'elle heure particulière il a disparu ce qui signifie qu'elle à été capturé.
- 2 - Le deuxiéme plan est d'analyser les details de ce paquet, si vous selectionnez un traffic particulier et les informations detaillées sur le traffic. Vous pourrez voir dans le deuxiéme pad. Donc la couche OSI signifie que c'est une trame qui indique comment les paquets passent d'une machine à une autre.

- **Ces 7 couches vous permettent de colorer la même chôse ici.**

  - a - La première est votre couche physique où vous touver toutes les informations, toutes les trames et combien de morsures ont été traversées.
  - b - La deuxième est votre couches de liaison des données où les données sont commutées en fonction de l'addresse MAC. Vous trouverez donc l'adresse MAC source, l'adresse MAC de destination.
  - c - La troisiéme est votre couche reseau où vous trouverez toutes les données et les données transmises en fonction du routage. Vous trouverez donc toutes toutes les adresses IP. Adresse IP source et Adresse IP destination.
  - d - La quatrième couche qui est la couche de transport où vous trouverez tous les services de PCP ou UDP et ajouterez leurs numéro de port associé. Port source et port de destination.

- 3 - Et celle ci est le paquet qui est visible sous la forme hexadécimale, ce sont donc les informations sur les paquet, que c'est sous la forme hexadécimale.

  ![Wireshark](/assets/01.png)
  ![Wireshark](/assets/02.png)
  ![Wireshark](/assets/03.png)
  ![Wireshark](/assets/04.png)
  ![Wireshark](/assets/05.png)

#### Colorisation du trafic

![Wireshark](/assets/06.png)
![Wireshark](/assets/07.png)
![Wireshark](/assets/08.png)

#### Port TCP et UDP

![Wireshark](/assets/10.png)
![Wireshark](/assets/11.png)
![Wireshark](/assets/12.png)
![Wireshark](/assets/13.png)
![Wireshark](/assets/14.png)
