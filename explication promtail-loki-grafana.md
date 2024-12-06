# explication promtail-loki-Grafana

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/ohlordhavemercy.png)

### installation et configuration de Promtail

Grafana nécessite des données centralisées pour les visualiser. Pour cela, on utilise **Promtail**, un agent de collecte compatible avec Grafana Loki.

installer promtail avec cette commande :

```bash
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
```

il faudra apres le deziper avec cette commande :

```bash
unzip promtail-linux-amd64.zip
```

il faudra apres le moove dans un autre répertoire ici :

```bash
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

puis lui donnée des droits :

```bash
sudo chmod +x /usr/local/bin/promtail
```

il faudra apres crée un fichiée dans /etc/promtail-config.yml

puis il faudra le nano et ecrire ceci dedans :

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

par la suite crée un service systemd pour gérer promtail avec cette commande :

```bash
sudo nano /etc/systemd/system/promtail.service
```

nano le pour y ajouter :

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

pour finir avec promtail recharger systemd et démarer promtail :

```bash
sudo systemctl daemon-reload
sudo systemctl start promtail
sudo systemctl enable promtail
```

c’est fini pour promtail maintenant le plus gros morceau loki.

### loki

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/naaaaaa.png)

first of all installer loki

```bash
wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
```

unzip loki:

```bash
unzip loki-linux-amd64.zip
```

bouge loki dans ce dossier:

```bash
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

donne les droit au dossier:

```bash
sudo chmod +x /usr/local/bin/loki
```

crée le fichiée de configuration de loki dans /etc/loki-config.yml

<aside>

>🚨ATTENTION selon la version de loki le script a ecrire dans ce dossier peux etre différent et vous renvoyer des erreur lors de la mise en route de celui-ci (ici la v11 de loki est utilisée)! 🚨                                                          

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

crée un service systemd pour loki :

```bash
sudo nano /etc/systemd/system/loki.service
```

nano le et ecris ceci dedans:

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

loki a besoin de certain dossier pour fonctionner correctement donc nous allons aussi les crée:

```bash
sudo mkdir -p /tmp/loki/index
sudo mkdir -p /tmp/loki/cache
sudo mkdir -p /tmp/loki/chunks
sudo chown -R root:root /tmp/loki
```

recharger le systemd et demarer loki:

```bash
sudo systemctl daemon-reload
sudo systemctl start loki
sudo systemctl enable loki
```

vous pouvez du coup voir sur http://localhost:9080 que votre target {job=”mumble”}

est bien rediriger de promtail a loki :

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana31.png)

la section portant sur loki est terminée ! (enfin)

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/faaaaaa.png)

passons maintenant a la configuration de Grafana!

### grafana

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/graaa.png)

prendre en main grafana n’est pas simple de prime abord mais apres une petite demi heure de prise en main vous comprendrez les bases et ce guide est la pour vous aidée.

deja commençons par installer le dépot officiel grafana: 

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

ensuite installer grafana:

```bash
sudo apt update
sudo apt install grafana -y
```

ensuite demarer et activée grafana:

```bash
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

une fois que c’est fait vous aurez acces a grafana sur votre navigateur via l’adresse http://localhost:3000 .

les identifiants par defaut sont admin/admin. A la premiere connexion grafana vous demandera de changer le mot de passe pour que les prochain acces soit plus sécurisée.

maintenant nous allons rajouter une source de data a grafana.

dans la barre de recherche ci dessous rechercher la categorie “data sources” et cliquer dessus.

![grafana1.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana1.png)

nous allons maintenant ajoutée la soucre de data loki a grafana pour qu’il puisse récoltée les donnée et le retranscrire sur grafana. selectionée add new data source. 

![grafana2.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana2.png?raw=true)

une fois dans le menu ci dessous tapper loki dans la barre de recherche et selectioner le.

![grafana3.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana3.png?raw=true)

![grafana4.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana4.png?raw=true)

vous aller donc atterire sur la page de configuration de la soucre data

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana30.png)

entrée votre url de connexion ici http://localhost:3100 .(selon la configuration de loki ci dessus si vous avez changer le port en faisant la configuration merci de renseigner le port que vous avez mis)

![grafana5.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana5.png?raw=true)

vous pouvez maintenant descendre tout en bas de la page et appuyer sur “save and test” pour sauvegarder et tester

![grafana6.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana6.png?raw=true)

une fois fait vous devriez avoir un message ressemblant a ceci pour vous dire que la sauvegarde a bien ete faite et que la source a ete correctement connecter.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana29.png)

bien la source ayant été ajoutée avec succes nous allon maintenant passer a la partit la plus marrante la creation du dash board. retourner donc au menu principal.Et selectionner le menu deroulant a gauche de votre ecrant avec trois barre horizontale.

![grafana7.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana7.png?raw=true)

selectionner dashboard.

![grafana8.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana8.png?raw=true)

crée un nouveau dashboard.

![grafana9.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana9.png?raw=true)

et ajoutée une nouvelle visualisation.

![grafana10.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana10.png?raw=true)

vous aller atterire sur une nouvelle page ou ont vous demande de choisir votre soucre de data pour la visualisation.donc séléctionner loki.

![grafana11.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana11.png)

un fois fait la petite fenetre va se fermer et vous aller etre sur le panneau de configurationde votre dashboard cliquer ici pour changer de visualisation.

![grafana12.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana12.png)

une fois fait le menu deroulant des visualisations va s’ouvrir ici nous voulons les logs donc nous allons simplement tapper logs dans la barre de recherche des visualisation.

![grafana13.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana13.png)

selectioner donc la visualisation logs.

![grafana14.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana14.png)

vous allez atterire sur une nouvelle visualisation celle des logs:

![grafana15.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana15.png)

1 : permet de changer le nom du dashboard (donc ici remplacer “panel title” par “logs”)

2 : permet d’ajoutée une description au dashboard

3 : permet de regler sur combien de temps nous voulons que la journalisation des logs s’effectue (ceci est changeable a tout moment meme apres la sauvegarde du dashboard)

4 : permet de refresh

nous allons maintenant entrée les requetes que nous voulons dans ce pannel.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana28.png)

comme vu plus haut promtail envoie a loki les info {job=”mumble”} et c’est ce que nous allons recupérer ici. entrée donc “job” et “mumble” dans le label filters.(comme vous pouvez le voir juste en dessous c’est exactement ce que nous cherchons).appuyer maintenant sur “run query”

![grafana16.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana16.png)

vous aurrez donc maintenant tout les logs sortant du mumble-server.log dirrectement dans le peaneau juste au dessus.

![grafana17.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana17.png)

mais ici nous avons vraiment tout ce qui se passe dans le .log nous ne voulons que les connections et les deconnexion nous allons donc copier coller les connexion (”Authenticated”) et les déconnexions (”Connexion closed”) dans la description pour pouvoir les recoller juste apres et faire notre description pas la meme occasion!

![grafana18.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana18.png)

maintenant nous allons afinnée les détails des logs que loki nous renvoie pour gardée uniquement les connexion est les deconnexion dans le serveur mumble.nous allons donc revenir unpeux plus bas dans nos requetes et ajouter une condition a l’affichage des ligne de log nous allons commencer par les connexion.copier coller depuis votre description “Authenticated” dans la catégorie “Line contains” de votre requete et réappuyer sur run query pour tout mettre a jour.

![grafana19.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana19.png)

maintenant dans votre panneau vous aurez uniquement les connexions au serveurs.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana32.png)

nous allons maintenant rajouter une requete avec une condition (”Connexion closed”) pour ajouter les deconexions a ce pannel.descendez donc tout en bas et ajouter une nouvelle requete en appuyant sur “add query”.

![grafana20.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana20.png)

nous allons du coup avoir une nouvelle petite fenetre de requete qui va s’ouvrir ou nous allons executer le meme prossec que pour la premiere requete mais a la place de “Authenticated” cette fois ci nous allons mettre la condition ”Connexion closed” que vous avez copier coller dans votre description vous pouvez donc la recopier et la coller dans l’emplacement indiquer et appuyer sur run queries une fois la requete remplie comme ce dessous.

![grafana21.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana21.png)

si votre pannel resemble désormait a ceci:

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana33.png)

c’est que vous avez réussi BRAVO!

si il ressemble a ceci:

![grafana22.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana22.png)

décocher l’option table view.

vous pouvez maintenant sauvegarder votre visualisation.

![grafana23.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana23.png)

et la nomée comme vous le souhaitez avec une autre description si vous le souhaitez.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana26.png)

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana27.png)

vous pouvez maintenant retourner au menu principal et aller dans votre menu deroulant pour selectioner votre categorie dashboard.une fois dedans votre dashboard “logs” devrait s’y trouvée:

![grafana25.PNG](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana25.PNG)

cliquer dessus pour avoir une vu de votre dashboard logs.

![image.png](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/grafana24.PNG)

et voila c’est tout je savais que tu allais y arrivée mon grand!

![oh oui.jpg](https://github.com/WildCodeSchool/TSSR-2411-P1-G2/blob/main/Ressources/oh%20oui.jpg)
