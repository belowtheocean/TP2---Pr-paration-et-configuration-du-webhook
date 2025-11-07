# TP2-Préparation-et-configuration-du-webhook

# Partie 1 — Préparation et configuration du webhook

Pour ce TP, j’ai créé un serveur Discord dédié aux alertes de sécurité.
Dans les paramètres du serveur → Intégrations → Webhooks, j’ai créé un nouveau webhook et j’ai récupéré son URL.
Ce webhook servira à recevoir automatiquement les alertes envoyées depuis la VM (fichiers sensibles lus, connexions SSH suspectes, etc).

URL : https://discord.com/api/webhooks/1436320908158435378/V8TN1tkCg81ax28LoMdpF8vY0RKBedkK8ZaMQVafULes75R8Yx7kSSoXgwDAJ1CU_rOn

# Partie 2 — Surveillance d’un fichier sensible + Alerte Discord

J’ai installé l’outil inotify-tools sur ma VM Ubuntu afin de surveiller en temps réel les accès à certains fichiers sensibles.

<img width="1919" height="461" alt="Capture d&#39;écran 2025-11-07 141240" src="https://github.com/user-attachments/assets/cf3d3bca-55c5-4480-8084-3b7898284294" />
<img width="1919" height="601" alt="Capture d&#39;écran 2025-11-07 141259" src="https://github.com/user-attachments/assets/34bbe76c-8366-48ab-80e3-1b8160862551" />

Ensuite, j’ai créé un fichier sensible /etc/secret.txt contenant des informations fictives, puis je l’ai protégé. ’ai ensuite créé un script bash qui utilise inotifywait pour détecter toute ouverture du fichier. Dès qu’un accès est détecté, le script envoie immédiatement une alerte dans le webhook Discord créé à l’étape 1. Puis j’ai rendu le script exécutable :

<img width="1919" height="72" alt="image" src="https://github.com/user-attachments/assets/cca5c724-a87d-454b-83b6-e99d1211e9cf" />

Enfin, j’ai lancé le watcher :

<img width="1919" height="39" alt="image" src="https://github.com/user-attachments/assets/b2469c35-0937-4a13-b39c-1b31b6254508" />

# Partie 3 — Surveillance des connexions SSH hors horaires

Dans cette partie, j’ai mis en place un système qui surveille les connexions SSH sur ma VM, uniquement hors des horaires de bureau. Le TP indiquait que les horaires de bureau sont entre 09h00 et 18h00.
Donc si quelqu’un se connecte en dehors de cette plage, cela doit déclencher une alerte via Discord (webhook déjà configuré à l’étape 1).

Pour ça :

j’ai créé un script Bash qui surveille en continu les logs d’authentification SSH (/var/log/auth.log)

et si une ligne indique “Accepted password” ou “Accepted publickey” en dehors de la plage horaire, je déclenche une alerte

Le script que j’ai créé (fichier /usr/local/bin/watch_ssh_hours.sh) :

<img width="1919" height="47" alt="image" src="https://github.com/user-attachments/assets/98666f61-d5c0-4665-9d4e-f0a43d51f517" />

<img width="1919" height="430" alt="Capture d&#39;écran 2025-11-07 152221" src="https://github.com/user-attachments/assets/e09fcfdf-d958-4f0f-aa2a-d950d6f5bd94" />

Puis je l’ai rendu exécutable :

<img width="1919" height="23" alt="image" src="https://github.com/user-attachments/assets/08a27152-e179-421c-bb6b-8edb49a227e6" />

J’ai ensuite lancé ce script en tâche active dans un terminal :

<img width="1919" height="27" alt="image" src="https://github.com/user-attachments/assets/5cd73377-6935-44cc-a000-81aa7098f422" />

Dans un autre terminal, je me suis reconnectée en SSH depuis Windows sur ma VM, volontairement hors horaires → ce qui a déclenché automatiquement l’alerte sur Discord.

<img width="1117" height="158" alt="image" src="https://github.com/user-attachments/assets/363f313d-929c-49c0-8c9e-a7f669c2c24a" />

# Partie 4 — Automatisation avec Cron

Pour terminer, j’ai configuré une automatisation avec cron pour que mes deux scripts de surveillance fonctionnent en continu, même si je ne suis pas connectée dans la VM.

Le script watch_secret.sh (surveillance du fichier /etc/secret.txt) doit tourner automatiquement au démarrage de la machine.

Le script watch_ssh_hours.sh (surveillance des connexions SSH hors horaires 9h-18h) doit être exécuté régulièrement toutes les 5 minutes.

Pour cela j’ai édité le crontab root :

<img width="1919" height="46" alt="image" src="https://github.com/user-attachments/assets/0acf86dc-c6d5-46d4-a07f-270496b1291a" />

Et j’ai ajouté ces deux lignes à la fin du fichier :
@reboot /usr/local/bin/watch_secret.sh
*/5 * * * * /usr/local/bin/watch_ssh_hours.sh

<img width="1919" height="673" alt="Capture d&#39;écran 2025-11-07 154016" src="https://github.com/user-attachments/assets/7554b18e-3329-4f6c-be27-a2a767162318" />

Ainsi, dès que la VM démarre, la surveillance du fichier sensible est active automatiquement. Et toutes les 5 minutes, la surveillance des connexions SSH hors horaires s’exécute en tâche de fond. Cette étape permet de rendre tout le système d’alertes autonome, sans intervention manuelle.

# Conclusion

Dans ce TP j’ai mis en place un système d’alertes de sécurité automatisé.

J’ai configuré un webhook Discord pour recevoir des notifications en cas d’accès à un fichier sensible ou de connexion SSH en dehors des horaires autorisés. Grâce à inotify-tools, journalctl et la planification via cron, la surveillance est maintenant continue et autonome, même après redémarrage de la VM. Ce fonctionnement permet d’augmenter la visibilité, d’anticiper des comportements suspects et de réagir plus rapidement face à un risque potentiel.
