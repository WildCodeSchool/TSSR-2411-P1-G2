# expliquation promtail+loki

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/a8f1abcd-45bd-4c7f-a21b-d38747c217ac/image.png)

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

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/3699449c-39bc-4e09-9678-01cdfe0c5367/image.png)

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
🚨

ATTENTION selon la version de loki le script a ecrire dans ce dossier peux etre différent et vous renvoyer des erreur lors de la mise en route de celui-ci (ici la v11 de loki est utilisée)! 🚨                                                          

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
🚨

Dans les version plus avancée de loki *(exemple: v13)* vous aurez besoin de rajoutée une ligne dans la section compactor 

```jsx
shared_store: filesystem
```

(juste en dessous de  “working_directory”  dans la section “compactor” *exemple:*

```jsx
compactor:
	working_directory: /tmp/loki/compactor
	shared_store: filesystem
```

dans cette version (ici la v11) seul le champ est requi “working_directory” et doit pointer vers un répertoire valide.

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

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/dd2cda06-381b-4d29-83c6-ddeb3818f366/image.png)

la section portant sur loki est terminée ! (enfin)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/4d5837cc-eaed-4f55-b692-acdf322986cf/image.png)

passons maintenant a la configuration de Grafana!

### grafana

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/612cf849-b5c3-417b-872e-98d4d47e10b4/image.png)

prendre en main grafana n’est pas simple de prime abord mais apres une petite demi heure de prise en main vous comprendre les bases et ce guide est la pour vous aidée.

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

![grafana1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/aa2be19f-8929-414a-9c33-0787711f6ded/grafana1.png)

nous allons maintenant ajoutée la soucre de data loki a grafana pour qu’il puisse récoltée les donnée et le retranscrire sur grafana. selectionée add new data source. 

![grafana2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/ff31269d-dab8-48f5-836b-a465d823a637/grafana2.png)

une fois dans le menu ci dessous tapper loki dans la barre de recherche et selectioner le.

![grafana3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/e9d46981-2871-4622-8bd2-4f8fa0b46feb/grafana3.png)

![grafana4.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/821b6a85-f2fd-45f4-b200-a03b3fb46ba5/grafana4.png)

vous aller donc atterire sur la page de configuration de la soucre data

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/f464c74a-9856-494b-8c7a-517df12b364f/image.png)

entrée votre url de connexion ici http://localhost:3100 .(selon la configuration de loki ci dessus si vous avez changer le port en faisant la configuration merci de renseigner le port que vous avez mis)

![grafana5.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/76fe5a89-a27f-4653-806c-5eb4c3ad8490/grafana5.png)

vous pouvez maintenant descendre tout en bas de la page et appuyer sur “save and test” pour sauvegarder et tester

![grafana6.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/1cfdb0f1-e526-4314-af52-3f20eb1a419b/grafana6.png)

une fois fait vous devriez avoir un message ressemblant a ceci pour vous dire que la sauvegarde a bien ete faite et que la source a ete correctement connecter.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/83c94423-8f93-4f39-b3ff-e313f797e035/image.png)

bien la source ayant été ajoutée avec succes nous allon maintenant passer a la partit la plus marrante la creation du dash board. retourner donc au menu [principal.Et](http://principal.Et) selectionner le menu deroulant a gauche de votre ecrant avec trois barre horizontale.

![grafana7.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/6bc24954-d8a2-4899-85ea-9fa3bb4fdde9/grafana7.png)

selectionner dashboard.

![grafana8.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/e52caae1-38dd-4425-926f-95a93ace2420/grafana8.png)

crée un nouveau dashboard.

![grafana9.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/9b6126cf-bcdd-43b9-bf7e-e94d6f861458/grafana9.png)

et ajoutée une nouvelle visualisation.

![grafana10.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/ff79af7a-ee48-4ebf-b090-733c2508c2ca/grafana10.png)

vous aller atterire sur une nouvelle page ou ont vous demande de choisir votre soucre de data pour la visualisation.donc séléctionner loki.

![grafana11.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/4f3c4bfd-46b1-4a88-b3cf-03f4ed572ee3/grafana11.png)

un fois fait la petite fenetre va se fermer et vous aller etre sur le panneau de configurationde votre dashboard cliquer ici pour changer de visualisation.

![grafana12.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/8289f576-ac31-449f-afde-edd0008c780c/grafana12.png)

une fois fait le menu deroulant des visualisations va s’ouvrir ici nous voulons les logs donc nous allons simplement tapper logs dans la barre de recherche des visualisation.

![grafana13.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/b07f9e3e-c8bd-48c0-885b-587634c6f417/grafana13.png)

selectioner donc la visualisation logs.

![grafana14.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/71714649-5955-4492-b45b-18603efcd7e0/grafana14.png)

vous allez atterire sur une nouvelle visualisation celle des logs:

![grafana15.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/d64ed76f-01c9-4be3-be7b-38e1c9b37ceb/grafana15.png)

1 : permet de changer le nom du dashboard (donc ici remplacer “panel title” par “logs”)

2 : permet d’ajoutée une description au dashboard

3 : permet de regler sur combien de temps nous voulons que la journalisation des logs s’effectue (ceci est changeable a tout moment meme apres la sauvegarde du dashboard)

4 : permet de refresh

nous allons maintenant entrée les requetes que nous voulons dans ce pannel.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/4ce20ae5-a159-4ef7-924e-e93cb67af08f/image.png)

comme vu plus haut promtail envoie a loki les info {job=”mumble”} et c’est ce que nous allons recupérer ici. entrée donc “job” et “mumble” dans le label filters.(comme vous pouvez le voir juste en dessous c’est exactement ce que nous cherchons).appuyer maintenant sur “run query”

![grafana16.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/775f57dd-8d35-42a4-a3d3-f1600f3ee331/grafana16.png)

vous aurrez donc maintenant tout les logs sortant du mumble-server.log dirrectement dans le peaneau juste au dessus.

![grafana17.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/93bcb7a6-5124-411a-8fe1-49cf19e6f9ca/grafana17.png)

mais ici nous avons vraiment tout ce qui se passe dans le .log nous ne voulons que les connections et les deconnexion nous allons donc copier coller les connexion (”Authenticated”) et les déconnexions (”Connexion closed”) dans la description pour pouvoir les recoller juste apres et faire notre description pas la meme occasion!

![grafana18.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/7a0b27ee-08d2-4797-9ee5-65f96dddaabc/grafana18.png)

maintenant nous allons afinnée les détails des logs que loki nous renvoie pour gardée uniquement les connexion est les deconnexion dans le serveur mumble.nous allons donc revenir unpeux plus bas dans nos requetes et ajouter une condition a l’affichage des ligne de log nous allons commencer par les connexion.copier coller depuis votre description “Authenticated” dans la catégorie “Line contains” de votre requete et réappuyer sur run query pour tout mettre a jour.

![grafana19.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/75db1c16-5d1f-46bb-9d6d-ce5a52b9654a/grafana19.png)

maintenant dans votre panneau vous aurez uniquement les connexions au serveurs.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/5aecaf57-fba0-4814-9c74-9df9d308aada/image.png)

nous allons maintenant rajouter une requete avec une condition (”Connexion closed”) pour ajouter les deconexions a ce pannel.descendez donc tout en bas et ajouter une nouvelle requete en appuyant sur “add query”.

![grafana20.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/4516beb5-f23c-41db-a678-49af15a885cc/grafana20.png)

nous allons du coup avoir une nouvelle petite fenetre de requete qui va s’ouvrir ou nous allons executer le meme prossec que pour la premiere requete mais a la place de “Authenticated” cette fois ci nous allons mettre la condition ”Connexion closed” que vous avez copier coller dans votre description vous pouvez donc la recopier et la coller dans l’emplacement indiquer et appuyer sur run queries une fois la requete remplie comme ce dessous.

![grafana21.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/a6831461-7f66-4af7-a90f-187b3bda137f/grafana21.png)

si votre pannel resemble désormait a ceci:

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/2fbf14e5-cc52-4555-bb71-421ee1bf39fb/image.png)

c’est que vous avez réussi BRAVO!

si il ressemble a ceci:

![grafana22.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/c08823c5-6b50-48e8-bf1b-e5362f998733/grafana22.png)

décocher l’option table view.

vous pouvez maintenant sauvegarder votre visualisation.

![grafana23.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/0cabc32a-fada-413f-acd4-c7766fc639a8/grafana23.png)

et la nomée comme vous le souhaitez avec une autre description si vous le souhaitez.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/fae6fa93-9f2d-4dc6-b4bc-0be559bd839c/image.png)

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/559b503d-bdec-459d-b225-3835014f2c42/image.png)

vous pouvez maintenant retourner au menu principal et aller dans votre menu deroulant pour selectioner votre categorie dashboard.une fois dedans votre dashboard “logs” devrait s’y trouvée:

![grafana25.PNG](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/51779809-2c83-4213-a796-cc1c479a5276/grafana25.png)

cliquer dessus pour avoir une vu de votre dashboard logs.

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/f3982fb2-36aa-4529-b53b-909070db11f7/image.png)

et voila c’est tout je savais que tu allais y arrivée mon grand!

![oh oui.jpg](https://prod-files-secure.s3.us-west-2.amazonaws.com/ef659afa-7e84-4c92-8471-907700355b88/67c5aa30-fd7a-46cc-8214-4a2e688fba44/oh_oui.jpg)
