# Enquête sur l'intégrité des fichiers à l'aide d'Auditd

#### Objectif

Surveiller l’intégrité des fichiers critiques (ex: `/etc/passwd`, `/etc/shadow`) pour détecter des modifications suspectes sur un hôte Linux via **Auditd** et l'intégrer à **ELK SIEM** pour l'analyse centralisée.

#### Installation d’Auditd

- **Auditd** est un service de journalisation de sécurité intégré à Linux, utilisé pour **surveiller l'intégrité des fichiers critiques et détecter toute action suspecte** sur un hôte. Dans le cadre de notre projet, il est couplé avec **ELK SIEM** pour visualiser et analyser les événements en temps réel, renforçant ainsi notre capacité à réagir rapidement en cas de tentative de compromission.

```sh
apt update
sudo apt install auditd audispd-plugins -y
systemctl enable auditd
systemctl start auditd
systemctl status auditd
```

- Vérification des logs

```sh
tail -f /var/log/audit/audit.log
```

#### Configuration d’Auditd

- Modifier les règles d’audi

```sh
nano /etc/audit/rules.d/audit.rules
```

- Contenu

```sh
## First rule - delete all
-D

## Increase the buffers to survive stress events.
## Make this bigger for busy systems
-b 8192

## This determine how long to wait in burst of events
--backlog_wait_time 60000

## Set failure mode to syslog
-f 1

## Surveillance des fichiers sensibles
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
```

- Redémarrer le service

```sh
systemctl restart auditd
```

#### Préparer ELK pour la détection

- Aller dans Kibana → `Management > Integrations`

- Rechercher Auditd Logs → Ajouter l’intégration

- Chemins : `/var/log/audit/audit.log*`

- Associer l’agent existant → Continuer → Valider

- Dashboards disponibles :

  - `[Logs Auditd] Audit Events`

![ELK](/Elastic_Stack_Ubuntu/assets/08.png)

![ELK](/Elastic_Stack_Ubuntu/assets/09.png)

#### Simuler l'attaque et visualiser les événements

- Simuler un changement dans /etc/passwd

```sh
# Ajouter un utilisateur
adduser linuxuser
passwd linuxuser

# Modifier manuellement le fichier (simulation attaque)
echo "testuser:x:1001:1001::/home/testuser:/bin/bash" >> /etc/passwd
```

#### Visualiser les logs

```sh
tail -f /var/log/audit/audit.log
ausearch -k passwd_changes
```

- Vérifier localement avec `ausearch`

- Visualiser dans Kibana `Analytics > Discover`

```sh
auditd.log.key:"passwd_changes"
```

![ELK](/Elastic_Stack_Ubuntu/assets/10.png)

```sh
message: "/etc/passwd"
```

![ELK](/Elastic_Stack_Ubuntu/assets/11.png)

#### Réponse à l'incident

- Supprimer la ligne manuelle ajoutée (ou commenter)

```sh
nano /etc/passwd

# Supprimer ou commenter :
# testuser:x:1001:1001::/home/testuser:/bin/bash
```
