# Linux Forensics

- Forensics Linux est le processus d’analyse d’un système basé sur Linux afin de détecter, enquêter et répondre à des incidents de sécurité.

#### Objectifs

- **Collecte de preuves**

- **Détection et réponse aux incidents**

- **Analyse approfondie**

- **Documentation et rapport**

#### Types de preuves

- **Collecte de données en direct** :

  - EDR (niveau entreprise)

  - Processus actifs : `ps, top, htop`

  - Réseau : `netstat, lsof -i`

  - Pile système

- **Capture de mémoire vive** :

  - `dd`, capture RAM, **LiME** (Linux Memory Extractor)

- **Image disque** :

  - `dd`

- **Analyse du système de fichiers** :

  - Métadonnées (`ls -l`), fichiers supprimés

- **Fichiers de logs** :

  - Logs système : `/var/log`

  - Logs applicatifs

- **Fichiers de configuration** :

  - `/etc/passwd`

- **Activité utilisateur**:

  - `.bash_history, .zsh_history`

- **Historique de connexion** :

  - `/var/log/lastlog, /var/log/auth.log`

- **Tâches planifiées** :

  - `/etc/, crontab`

#### 1 - Analyse des logs système avec journalctl

> `journalctl` permet d’afficher et de filtrer les journaux du système `systemd`.

```sh
journalctl -n 100
journalctl --since "1 hour ago"
journalctl -u ssh
journalctl | grep "error"
journalctl -b > logs_from_last_boot.txt
```

#### 2 - Analyse de fichiers logs avec awk

- `awk` est un langage de traitement de texte utile pour extraire des champs spécifiques dans les logs.

```sh
nano /var/log/syslog
awk '{print $1, $2, $3}' /var/log/syslog
awk '/root/ {print $0}' /var/log/syslog
awk '/root/ {count++} END {print count}' /var/log/auth.log
awk '/failed/ {count++} END {print count}' /var/log/auth.log
awk '/session opened/ {user[$11]++} END {for (u in user) print u, user[u]}' /var/log/auth.log
awk '/root/' /var/log/syslog > warning_logs.txt
nano warning_logs.txt
```

#### 3 - Suivi en temps réel avec tail

- `tail` affiche la fin d’un fichier, utile pour la surveillance en direct des logs.

```sh
tail /var/log/syslog
tail -f /var/log/syslog | grep --color=auto "error"
tail -f /var/log/syslog /var/log/auth.log
tail -n 50 -f /var/log/syslog
```

#### 4 - Résumé de logs avec logwatch

- `logwatch` génère des rapports résumés à partir des logs du système.

```sh
sudo apt-get install logwatch
logwatch --help
sudo logwatch --detail low --range today
```

#### 5 - Audit des événements système avec auditd

- `auditd` permet de suivre les événements sensibles du système à l’aide de règles personnalisées.

```sh
sudo apt-get install auditd
sudo systemctl start auditd
sudo systemctl status auditd
sudo auditctl -w /bin/chmod -p x -k chmod_changes
sudo chmod 755 warning_logs.txt
sudo ausearch -k chmod_changes
```

#### Analyse du système de fichiers Linux

- C’est le processus d’examen de la structure et du contenu d’un système de fichiers pour découvrir et analyser des preuves numériques.

- **Objectifs** :

  - Reconstituer les activités des utilisateurs
  - Identifier les intrusions, récupérer des données supprimées
  - Fournir des preuves pour des enquêtes juridiques ou des réponses aux incidents

#### Activités d’analyse

- **Fichiers et répertoires** :

  - Attributs (`lsattr, chattr`), horodatage (`stat`), fichiers cachés (`ls -a, find / -name ".*"`)

- **Examen des métadonnées**

- **Analyse de contenu** :

  - Recherche d’injection (`grep`, **YARA**)

  - Analyse de malwares (ClamAV, VirusTotal, strings)

  - Données chiffrées AES-256

- **Activités avancées** :

  - Récupération de données, détection de malwares

#### Système de fichiers Linux

- Un système de fichiers Linux est une méthode structurée pour stocker, organiser et gérer les données sur un support.

#### Types :

- `Ext2, Ext3, Ext4, ReiserFS, XFS, Btrfs, etc.`

#### Structure :

- `Superbloc, table des inodes, blocs de données, répertoires, métadonnées`

#### Répertoires importants

- `/home/username`
- `/` (racine)
- `/etc/network/interfaces`
- `/etc/hosts`
- `/etc/passwd`
- `/etc/shadow`
- `/var/log`
- `/var/www`

#### Outils courants

- `ls` - Lister les fichiers
- `stat` - Voir les métadonnées d’un fichier
- `find` - Rechercher des fichiers
- `grep` - Rechercher du texte
- `file` - Déterminer le type de fichier
- `df` - Utilisation des disques
- `du` - Estimation de l’espace utilisé
- `mount` - Monter/démonter des systèmes de fichiers

#### 1 - Lister le contenu d’un répertoire

```sh
ls -l /home
ls -la /home
```

#### 2 - Analyser les métadonnées d’un fichier

```sh
stat /home/username/file.txt
file /home/username/file.txt
find /home -name "*.txt"
find /home -empty
find /home -type f -size +100M
find /home -type f -mtime -7
find /home -type f -atime -7
find /home -type f -ctime -7
```

#### 3 - Analyser l’utilisation du disque

```sh
df -h
du -sh /home/username
du -ah /home/username
```

#### 4 - Rechercher du contenu dans des fichiers

```sh
echo "This is a test file for malware analysis." > test.exe
grep "malware" *.exe
grep -r "malware"
grep -c "malware" *.exe
```
