# Analyse de la Mémoire (Memory Forensics)

> L’analyse de la mémoire est le processus d’analyse de la mémoire volatile (RAM) d’un ordinateur afin d’extraire des informations critiques, des artefacts et des éléments relatifs aux activités système, incidents de sécurité et enquêtes numériques.

> Elle fournit une image en temps réel de l’état du système, permettant de détecter des malwares, de révéler des rootkits et de reconstituer les activités des utilisateurs pour la réponse aux incidents et l’investigation numérique.

#### Avantages de l’analyse de la mémoire

- Cela est utile en raison de la manière dont les processus, fichiers et programmes s’exécutent en mémoire. Une fois un instantané capturé, de nombreuses informations importantes peuvent être obtenues par l’enquêteur, comme :

  > Les processus en cours d’exécution

  > Les fichiers exécutables actifs

  > Les ports ouverts, adresses IP et autres informations réseau

  > Les utilisateurs connectés au système, et depuis où

  > Les fichiers ouverts et par qui

#### Données Volatiles

> En analyse de mémoire, les données volatiles se réfèrent aux informations stockées dans la mémoire vive (RAM) d’un ordinateur, qui sont temporaires et perdues lorsque le système est éteint ou redémarré.

> Les données volatiles s’opposent aux données non-volatiles, stockées sur des périphériques de stockage permanents (comme les disques durs) et qui restent intactes même après l’arrêt du système.

> Les données volatiles doivent être bien comprises par tout intervenant en réponse aux incidents, car l’une des premières réactions face à un appareil compromis peut être de l’éteindre pour contenir la menace.

> Le problème avec cette approche est que tout malware en cours d’exécution est probablement en mémoire. Ainsi, les connexions réseau et processus actifs seront perdus car ces informations, résidant uniquement en mémoire, disparaissent à l’arrêt.

#### Capture de la mémoire (Memory Dump)

> Un **dump mémoire** (ou capture de la RAM) est un instantané de la mémoire capturé à des fins d’analyse. Il contient les données relatives à tous les processus en cours au moment de la capture.

#### Acquisition de la mémoire avec FTK Imager

- [https://archive.org/download/access-data-ftk-imager-4.7.1.](https://archive.org/download/access-data-ftk-imager-4.7.1)

#### Volatility

> `Volatility` est un framework open-source d’analyse de la mémoire utilisé dans les enquêtes numériques pour analyser la mémoire volatile (RAM).

> `Objectif` : extraire des artefacts numériques tels que les processus actifs, connexions réseau et pilotes chargés à partir de dumps mémoire.

> `Fonctionnalités` : propose de nombreux plugins pour analyser des images mémoire provenant de plusieurs systèmes d’exploitation (Windows, Linux, MacOS, etc.).

> `Communauté` : Volatility dispose d’une grande communauté d’utilisateurs et de contributeurs qui développent des plugins, partagent leurs connaissances et offrent du support.

#### Composants clés de Volatility

> `Noyau de Volatility` : fournit les fondations de l’analyse mémoire, incluant la traduction d’adresses mémoire, la liste des processus et la gestion des plugins.

> `Plugins` : outils spécialisés extrayant des informations spécifiques, comme pslist pour les processus, netscan pour l’analyse réseau, et filescan pour l’exploration du système de fichiers.

#### Installation de Volatility

> [https://volatilityfoundation.org/the-volatility-framework/.](https://volatilityfoundation.org/the-volatility-framework/)

> [https://github.com/volatilityfoundation/volatility3/releases/tag/v2.5.0](https://github.com/volatilityfoundation/volatility3/releases/tag/v2.5.0)

```sh
uzip volatility3-2.5.0
cd /opt/
cd volatility/volatility3-2.5.0/
python3 vol.py --help
python3 vol.py -f memdump.mem windows.vadinfo.VadInfo
```

#### 1 - Récupérer les informations de l’image mémoire

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.info
```

#### 2 - Collecter les informations sur les processus

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.pslist
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.psscan.PsScan
```

#### 3 - Récupérer la ligne de commande des processus

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.cmdline
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.cmdline --pid 2580
```

#### 4 - Collecte des informations réseau

```sh
python3 /opt/volatility/volatility3-2.5.0/vol.py -f memdump.mem windows.netscan
```

- [https://www.virustotal.com/gui/home/search.](https://www.virustotal.com/gui/home/search)
