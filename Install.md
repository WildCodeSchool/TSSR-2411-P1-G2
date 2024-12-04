# install.md
---
> Documentation à destination des développeurs pour l'installation et la configuration du serveur.



## **Configuration du server-mumble.ini**
`sudo nano /etc/mumble-server.ini`
![](https://media.discordapp.net/attachments/1311982568035323915/1311985009233498124/mumble_serv.png?ex=674ad8c1&is=67498741&hm=7bf5d1477de7808b8ddc23f88ea330721df28a7fbd0cbc6844deffdba7a6732b&=&format=webp&quality=lossless&width=658&height=565)

**1.  Modification du port de connexion au serveut mumble**
**2. Modification de mot de passe pour se connecter au serveur Mumble**
**3. Maximum d'utilisateur autoriser en meme temps sur le serveur**
**4. utilisateur maximum par channel (0 etant une no limit)**

---
## **Configuration du serveur ssh**
`sudo nano /etc/ssh/sshd_config`
![](Ressources/portssh.png)
>dans l'encart présentée ci-dessus vous pouvez modifier le port de connexion au serveur ssh 
---


![](https://media.discordapp.net/attachments/1311982568035323915/1311985817555828747/mumbleserv2.png?ex=674ad982&is=67498802&hm=5d6fb10f1c2ada218b34b99fdea8334bba362bd72c8d3caf3d5af399b36ed195&=&format=webp&quality=lossless&width=621&height=565)
**5. Choix du salon par defaut lors de la permiere entrée dans le serveur**

---
## **Configuration mumble-server.log**
`root@flosh:~# sudo nano /var/log/mumble-server/mumble-server.log`
![](https://media.discordapp.net/attachments/1311982568035323915/1311991178756886628/mumbleservelog.png?ex=674ade80&is=67498d00&hm=64d877622044339176754d66f83d9b85f157686f5a76c8e6d268f461f9c0442c&=&format=webp&quality=lossless&width=1345&height=565)

**1. Configuration du mot de passe du SuperUser**
>Si le mot de passe a été changer, que vous ne pouvez pas vous connecter et que l'ancien mot de passe n'a pas été écrit quelque part utilisée cette commande :
`sudo murmurd -supw <pw>`
`sudo murmurd -ini /etc/mumble-server.ini`
Remplacer `<pw>` par ce que vous désirez mettre en mot de passe pour l'utilisateur "SuperUser".

Si vous decendez plus bas vous pouvez voir les "logs" du serveur.
![](https://media.discordapp.net/attachments/1311982568035323915/1311991903495127060/mumbleserverlog2.PNG?ex=674adf2d&is=67498dad&hm=164402911ca88a6bcb0245a61db69e07e296a353d6ea23da6d35a6bd60c63010&=&format=webp&quality=lossless&width=972&height=563)
---
## Mumble-server.sqlite
`root@flosh:~# sudo nano /var/lib/mumble-server.sqlite`
![](https://media.discordapp.net/attachments/1311982568035323915/1311992733371465782/image.png?ex=674adff3&is=67498e73&hm=ff1d5c087c0a105ba0a6cf82d0eab09e41b6eafc0979de259cfe341029ab4f35&=&format=webp&quality=lossless&width=1423&height=563)
>**Clé privée RSA**
La clé privée RSA correspond à la clé secrète associée au certificat public.
Elle est utilisée pour :
Déchiffrer les messages chiffrés par le client avec la clé publique.
Signer des communications, garantissant qu'elles proviennent du serveur authentique.
**Important :** Cette clé doit être gardée secrète. Si elle est compromise, un tiers pourrait intercepter ou usurper les communications.

![](https://media.discordapp.net/attachments/1311982568035323915/1311992890217463838/image.png?ex=674ae018&is=67498e98&hm=e744a2a4657a03c7619b26e8ab70b9f1ad892b114972ca33a174089c3c905c5a&=&format=webp&quality=lossless&width=1440&height=460)
>**Certificat RSA**
Le certificat RSA est un fichier contenant une clé publique qui identifie le serveur de manière sécurisée.
Il est utilisé pour établir des connexions chiffrées entre les clients Mumble et le serveur Murmur.
Lorsqu'un client se connecte, le serveur utilise ce certificat pour prouver son identité et permettre le chiffrement de la communication.

Ce fichiée contient aussi :
* 1.Utilisateurs et Identifiants
    * Liste des utilisateurs enregistrés.
    * Informations d'identification des utilisateurs : noms, certificats, mots de passe hachés (si activés).
    * Statistiques sur les connexions (nombre de connexions, dernières connexions, etc.).
* 2.Canaux
    * Structure des canaux créés sur le serveur.
    * Permissions et hiérarchies spécifiques aux canaux.
* 3.Permissions et Groupes
    * Droits d'accès pour les utilisateurs ou groupes à différents canaux.
    * Groupes personnalisés et leurs permissions associées.
* 4.Banlist
    * Liste des utilisateurs bannis, basée sur leur IP ou certificat.
    * Raison et durée des bannissements.
* 5.Configuration spécifique au serveur
    * Paramètres persistants spécifiques à l'instance du serveur.
    * Préférences définies via l'interface ou les commandes administratives.
* 6.Logs
    * Journaux d'événements (selon la configuration, les logs peuvent être stockés ici).

![](https://media.discordapp.net/attachments/1311982568035323915/1311994602386886749/mumble-serversqlite2.png?ex=674ae1b0&is=67499030&hm=9517472830b7ba3f7881c578b025d469d9a7a35f5c543cc4355f2205a1ebf10f&=&format=webp&quality=lossless&width=967&height=565)

---
