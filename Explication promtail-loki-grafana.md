# Explication Promtail-Loki-Grafana

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/ohlordhavemercy.png)

## Installation et configuration de PROMTAIL

Grafana nécessite des données centralisées pour les visualiser. Pour cela, on utilise **Promtail**, un agent de collecte compatible avec Grafana Loki.

Installer Promtail avec cette commande :

```bash
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
```

Il faudra après le deziper avec cette commande :

```bash
unzip promtail-linux-amd64.zip
```

Il faudra après le déplacer dans un autre répertoire ici :

```bash
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

Puis lui donner des droits :

```bash
sudo chmod +x /usr/local/bin/promtail
```

Il faudra après crée un fichié dans /etc/promtail-config.yml

puis il faudra l'ouvrir avec un éditeur de texte (nano ou vim) et écrire ceci dedans :

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/log/promtail-positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: mumble_logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: "mumble"
          __path__: /var/log/mumble-server/mumble-server.log
```

Par la suite créez un service systemd pour gérer Promtail avec cette commande :

```bash
sudo nano /etc/systemd/system/promtail.service
```

Ouvrez le avec un éditeur de texte pour y ajouter :

```
[Unit]
Description=Promtail service
After=network.target

[Service]
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail-config.yml
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Pour finir avec Promtail rechargez systemd et démarrez Promtail :

```bash
sudo systemctl daemon-reload
sudo systemctl start promtail
sudo systemctl enable promtail
```

**C’est fini pour promtail maintenant le plus gros morceau Loki.**

## LOKI

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/naaaaaa.pn)

Télécharger Loki depuis ce dépot Github :

```bash
wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
```

Unzip le dossier télécharger :

```bash
unzip loki-linux-amd64.zip
```

Déplacez Loki dans ce dossier:

```bash
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

Donnez les droits au dossier:

```bash
sudo chmod +x /usr/local/bin/loki
```

Créez le fichié de configuration de loki dans /etc/loki-config.yml

<aside>

>🚨ATTENTION selon la version de loki le script à écrire dans ce dossier peux etre différent et vous renvoyer des erreur lors de la mise en route de celui-ci (ici la v11 de loki est utilisée)! 🚨                                                          

</aside>

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 5m
  chunk_retain_period: 30s

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/index
    cache_location: /tmp/loki/cache
  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  allow_structured_metadata: false

compactor:
  working_directory: /tmp/loki/compactor
```

<aside>

🚨Dans les version plus avancée de loki *(exemple: v13)* vous aurez besoin de rajoutée une ligne dans la section compactor 

```jsx
shared_store: filesystem
```

(juste en dessous de  “working_directory”  dans la section “compactor” *exemple:*

```jsx
compactor:
	working_directory: /tmp/loki/compactor
	shared_store: filesystem
```

dans cette version (ici la v11) seul le champ est requi “working_directory” et doit pointer vers un répertoire valide.🚨

</aside>

Créez un service systemd pour Loki :

```bash
sudo nano /etc/systemd/system/loki.service
```

Ouvrez le avec un éditeur de texte et écrivez ceci dedans:

```
[Unit]
Description=Loki service
After=network.target

[Service]
ExecStart=/usr/local/bin/loki -config.file=/etc/loki-config.yml
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Loki a besoin de certains dossiers pour fonctionner correctement donc nous allons aussi les créer:

```bash
sudo mkdir -p /tmp/loki/index
sudo mkdir -p /tmp/loki/cache
sudo mkdir -p /tmp/loki/chunks
sudo chown -R root:root /tmp/loki
```

Rechargez le systemd et démarrez loki:

```bash
sudo systemctl daemon-reload
sudo systemctl start loki
sudo systemctl enable loki
```

Vous pouvez du coup voir sur http://localhost:9080 que votre target {job=”mumble”}

est bien rediriger de promtail a loki :

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana31.png)

**La section portant sur loki est terminée ! (enfin)**

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/faaaaaa.png)

passons maintenant a la configuration de Grafana!

## GRAFANA

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/graaa.png)

Prendre en main grafana n’est pas simple de prime abord mais après une petite demi heure de prise en main vous comprendrez les bases et ce guide est la pour vous aider.

Déja commençons par installer le dépot officiel grafana: 

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

Ensuite installez Grafana:

```bash
sudo apt update
sudo apt install grafana -y
```

Ensuite démarrez et activez Grafana:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Une fois que c’est fait vous aurez acces a Grafana sur votre navigateur via l’adresse http://localhost:3000 .

Les identifiants par defaut sont **admin/admin**. A la premiere connexion Grafana vous demandera de changer le mot de passe pour que les prochains accès soit plus sécurisés.

Maintenant nous allons rajouter une source de data à Grafana.

Dans la barre de recherche ci dessous recherchez la categorie “data sources” et cliquez dessus.

![grafana1.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana1.png)

Nous allons maintenant ajouter la soucre de data loki a Grafana pour qu’il puisse récolter les données et les retranscrire sur Grafana. 

Selectionnez add new data source. 

![grafana2.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana2.png?raw=true)

Une fois dans le menu ci dessous tappez loki dans la barre de recherche et selectionez le.

![grafana3.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana3.png?raw=true)

![grafana4.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana4.png?raw=true)

Vous allez donc atterire sur la page de configuration de la soucre data

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana30.png)

Entrez votre url de connexion ici http://localhost:3100 .(selon la configuration de loki ci dessus, si vous avez changer le port en faisant la configuration merci de renseigner le port que vous avez mis)

![grafana5.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana5.png?raw=true)

Vous pouvez maintenant descendre tout en bas de la page et appuyer sur “save and test” pour sauvegarder et tester

![grafana6.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana6.png?raw=true)

Une fois fait vous devriez avoir un message ressemblant a ceci pour vous dire que la sauvegarde a bien été faite et que la source a été correctement connectée.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana29.png)

Bien la source ayant été ajoutée avec succes nous allons maintenant passer a la partie la plus marrante, la creation du Dash-board. 
Retournez donc au menu principal.
Et selectionnez le menu deroulant à gauche de votre écran avec trois barre horizontale.

![grafana7.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana7.png?raw=true)

Selectionnez Dashboard.

![grafana8.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana8.png?raw=true)

Créez un nouveau Dashboard.

![grafana9.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana9.png?raw=true)

et ajoutez une nouvelle visualisation.

![grafana10.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana10.png?raw=true)

Vous allez atterrir sur une nouvelle page ou on vous demande de choisir votre source de data pour la visualisation.
Donc séléctionnez Loki.

![grafana11.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana11.png?raw=true)

Une fois fait la petite fenetre va se fermer et vous allez étre sur le panneau de configuration de votre Dashboard. 
Cliquez ici pour changer de visualisation.

![grafana12.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana12.png?raw=true)

Une fois fait le menu deroulant des visualisations va s’ouvrir ici nous voulons les logs donc nous allons simplement tapper logs dans la barre de recherche des visualisation.

![grafana13.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana13.png?raw=true)

Selectionez donc la visualisation logs.

![grafana14.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana14.png?raw=true)

Vous allez atterrir sur une nouvelle visualisation, celle des logs:

![grafana15.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana15.png?raw=true)

1 : permet de changer le nom du dashboard (donc ici remplacer “panel title” par “logs”)

2 : permet d’ajouter une description au dashboard

3 : permet de régler sur combien de temps nous voulons que la journalisation des logs s’effectue (ceci est modifiable à tout moment meme après la sauvegarde du dashboard)

4 : permet de refresh

Nous allons maintenant entrer les requetes que nous voulons dans ce pannel.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana28.png)

Comme vu plus haut Promtail envoie a Loki les info {job=”mumble”} et c’est ce que nous allons recupérer ici. 
Entrez donc “job” et “mumble” dans le label filters. (comme vous pouvez le voir juste en dessous c’est exactement ce que nous cherchons). 
Appuyer maintenant sur “run query”

![grafana16.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana16.png)

Vous aurez donc maintenant tout les logs sortant du mumble-server.log directement dans le panneau juste au dessus.

![grafana17.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana17.png)

Mais ici nous avons vraiment tout ce qui se passe dans le .log nous ne voulons que les connections et les déconnexions nous allons donc copier-coller les connexions (”Authenticated”) et les déconnexions (”Connexion closed”) dans la description pour pouvoir les recoller juste après et faire notre description pas la meme occasion!

![grafana18.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana18.png)

Maintenant nous allons affinner les détails des logs que Loki nous renvoie pour garder uniquement les connexions et les déconnexion dans le serveur mumble. 
Nous allons donc revenir un peux plus bas dans nos requetes et ajouter une condition a l’affichage des ligne de log nous allons commencer par les connexion. 
Copiez-collez depuis votre description “Authenticated” dans la catégorie “Line contains” de votre requete et réappuyez sur run query pour tout mettre a jour.

![grafana19.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana19.png)

Maintenant dans votre panneau vous aurez uniquement les connexions au serveurs.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana32.png)

Nous allons maintenant rajouter une requete avec une condition (”Connexion closed”) pour ajouter les deconexions a ce pannel. 
Descendez donc tout en bas et ajoutez une nouvelle requete en appuyant sur “add query”.

![grafana20.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana20.png)

Nous allons du coup avoir une nouvelle petite fenetre de requete qui va s’ouvrir ou nous allons executer le meme prossesus que pour la premiere requete mais a la place de “Authenticated” cette fois ci nous allons mettre la condition ”Connexion closed” que vous avez copié-collé dans votre description vous pouvez donc la recopier et la coller dans l’emplacement indiqué et appuyer sur run queries une fois la requete remplie comme ci-dessous.

![grafana21.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana21.png)

Si votre pannel ressemble désormait a ceci:

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana33.png)

C’est que vous avez réussi BRAVO!

Si il ressemble a ceci:

![grafana22.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana22.png)

Décochez l’option table view.

Vous pouvez maintenant sauvegarder votre visualisation.

![grafana23.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana23.png)

et la nommer comme vous le souhaitez avec une autre description si vous le souhaitez.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana26.png)

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana27.png)

Vous pouvez maintenant retourner au menu principal et aller dans votre menu deroulant pour selectioner votre categorie Dashboard.
Une fois dedans votre dashboard “logs” devrait s’y trouvée:

![grafana25.PNG](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana25.PNG)

Cliquez dessus pour avoir une vu de votre dashboard logs.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana24.PNG)

Et voila c’est tout, je savais que tu allais y arriver mon grand!

![oh oui.jpg](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/oh%20oui.jpg)
