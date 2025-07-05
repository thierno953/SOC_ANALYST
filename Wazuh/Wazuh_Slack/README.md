# Intégrer Wazuh avec Slack en utilisant l'API externe de Wazuh pour envoyer des alertes en temps réel dans un canal Slack

#### Slack:

- Slack est une plate-forme de collaboration basée sur le cloud qui facilite la communication et le travail d'équipe au sein de l'organisation.
- L'integration utilise le Web Hook entrant de Slack et permet aux professionnels de la sécurité de recevoir des alertes en temps réel directement dans les canaux désignés.

- Pour configurer l'intégration, effectuez les étapes suivantes:

  - Activez le Web Hook entrant, créez l'un des canaux Slack, suivez lz guide Slack sur le Web entrant.

- Resource: [https://documentation.wazuh.com/current/user-manual/manager/integration-with-external-apis.html](https://documentation.wazuh.com/current/user-manual/manager/integration-with-external-apis.html)

- Nous allons d'abord implémenter cela, nous allons créer notre canal Slack en utilisant ce guide.

![Slack](/Wazuh/Wazuh_Slack/Wazuh_Slack/00.png)

- Envoi de messages à l'aide du Web Hook entrant. Premiers pas avec le Web Hook entrant. Créez une application Slack

![Slack](/Wazuh/Wazuh_Slack/01.png)

![Slack](/Wazuh/Wazuh_Slack/02.png)

- Connecter ou nous allons créer un compte à cet effet vous avez besoin d'un email professionel ou d'une adresse e-mail d'entreprise. Je vais donc utiliser mon adresse Gmail

![Slack](/Wazuh/Wazuh_Slack/03.png)

- Nous avons reçu l'e-mail de confirmation sur notre cette adresse e-mail

![Slack](/Wazuh/Wazuh_Slack/04.png)

![Slack](/Wazuh/Wazuh_Slack/05.png)

![Slack](/Wazuh/Wazuh_Slack/06.png)

- Nous allons créer un nom d'espace de travail avec Wazuh

![Slack](/Wazuh/Wazuh_Slack/07.png)

![Slack](/Wazuh/Wazuh_Slack/08.png)

- Je vais ajouter de collègue par e-mail

![Slack](/Wazuh/Wazuh_Slack/09.png)

![Slack](/Wazuh/Wazuh_Slack/10.png)

- Nous devons créer notre application Slack

![Slack](/Wazuh/Wazuh_Slack/11.png)

![Slack](/Wazuh/Wazuh_Slack/12.png)

![Slack](/Wazuh/Wazuh_Slack/13.png)

- Activer le WebHook entrant, vous serez redirigé vers la page de configuration de votre nouvelle application ou si vous utilisez une application existante tableau de bord de gestion des applications à partir d'ici sélectionnez les webhooks entrants

![Slack](/Wazuh/Wazuh_Slack/14.png)

![Slack](/Wazuh/Wazuh_Slack/15.png)

- Activer les webhooks entrants

![Slack](/Wazuh/Wazuh_Slack/16.png)

- Ajouter un nouveau webhook à l'espace d travail

![Slack](/Wazuh/Wazuh_Slack/17.png)

- Choisissez un canal sur lequel l'application publiera pour sélectionner, autoriser si vous devez ajouter le webhook entrant à un canal privé

![Slack](/Wazuh/Wazuh_Slack/18.png)

- Maintenant si vous devez ajouter le webhook entrant à un canal privé vous devez d'abord être dans ce canal vous serez renvoyé aux paramètres de l'application où devriez voir une nouvelle entrée sous l'URL du webhook, votre URL webhook ressemblera à quelque chose comme ça

![Slack](/Wazuh/Wazuh_Slack/19.png)

- Ajoutez la configuration ci-dessus à ce ficher `URL` sur le serveur Wazuh, remplacez l'URL du webhook par votre webhook entrant.

```sh
nano /var/ossec/etc/ossec.conf
```

```sh
<ossec_config>
  <integration>
    <name>slack</name>
    <hook_url>https://hooks.slack.com/services/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</hook_url>
    <level>10</level> <!-- Seules les alertes niveau 10 seront envoyées -->
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

- Protégez le fichier ossec.conf avec des permissions strictes

```sh
chmod 600 /var/ossec/etc/ossec.conf
chown root:ossec /var/ossec/etc/ossec.conf
```

```sh
service wazuh-manager restart
```

![Slack](/Wazuh/Wazuh_Slack/20.png)

#### Tester le fonctionnement

- Exécute ce script pour déclencher une alerte manuellement :

```sh
/var/ossec/bin/manager_control -l 10 -m "Test Slack Alert from Wazuh"
```

- Tu devrais voir le message apparaître dans ton canal Slack dans les secondes qui suivent.
