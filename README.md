# TSSR-2411-P1-G2
# **Projet: Serveur de communication vocal**
---
## **Introduction: Mise en contexte**
Ce projet a pour objectif la mise en place d'une infrastructure permettant des communications vocales sécurisées entre plusieurs clients via le logiciel open source **Mumble**. 
Cela inclut :
* Configuration d’un serveur vocal **Mumble** sur un système **Debian 12.**
* Connexion de deux clients à ce serveur, avec des systèmes d’exploitation variés (Windows, Linux, macOS, etc.).
* Garantir que les communications soient fonctionnelles et sécurisées.

### **Objectif secondaire** :
1. Journalisation :
* Suivi des activités des utilisateurs (heures de connexion, messages, etc.).
* Enregistrement des journaux d'accès pour analyser les incidents.
2. Surveillance des sessions :
* S'assurer que le serveur reste actif et performant.
* Utiliser des outils de monitoring pour vérifier les connexions et détecter des anomalies.
* Mettre en place une journalisation et une surveillance des sessions.

### **Ressources nécessaires**
1. Serveur : Une machine **Debian 12** (physique ou virtuelle).
2. Logiciel serveur **Mumble** :
3. Installation de **Murmur** (le serveur de Mumble).
4. Clients :
* Deux ordinateurs ou machines virtuelles exécutant des systèmes variés (Windows, macOS, Linux) avec le logiciel client Mumble.
5. Outils de monitoring :
* Logiciels: **Prometheus**, et "Grafana" pour surveiller et journaliser les sessions.

## **Membres du Groupe de Projet et Rôles**
Scrum Master
Barbara, experte dans la rédaction, maitrise le Markdown, bilingue (on prévoit un déploiement en Italie), comprend aussi bien les enjeux que la technique.
Product Owner
Saddem, très bon porte-parole grâce à sa voix qui impose le respect, apporte une grande aide aux développeurs avec sa connaissance approfondie de la programmation et du réseau.
Développeurs
Alexandre (Flosh) , try-harder de la programmation, son repas préféré : une grosse assiette de configuration.ini saupoudrée de documentation.md.
Julien, navigue à vue entre ses vms sur Proxmox, surf sur les scripts et apprécie les pool d’adresse IP.



Sprint 1
Comprendre le fonctionnent du logicielle
Analyser l’effet des modifications du fichier de configuration
Résoudre les 1er problèmes techniques
Création des VM , installation des logicielles et configuration

SPRINT 2
Ajout de l’interface web de Mumble
Ajout d’une solution de monitoring de logs
Rédaction de la documentation claire et détaillée
Optimisation, ajout de fonctionnalités et rédaction de la documentation




# **Prérequis Techniques**
---
## Matériel
* Ordinateur
## Logiciel
* Systèmes d'exploitation Debian 12.iso.
* Server Mumble.
* VMware, Virtual Box ou Proxmox.
* Un accès SSH au serveur Mumble une fois installé.

### **Étapes Détaillées du Projet**:
**1. Installation du Serveur Mumble sur Debian 12**
- **Mise à jour du système:** S'assurer que tous les paquets sont à jour.
- **Installation de Mumble:** Utiliser le gestionnaire de paquets `apt` pour installer Mumble et ses dépendances.
- **Configuration:** Configurer le serveur pour définir les ports, les canaux, les autorisations, etc.

**2. Configuration de la Journalisation**
- **Choix d'un outil:** Utiliser des outils comme `syslog`, `rsyslog`, ou des solutions plus spécialisées comme `auditd`.
- **Configuration:** Configurer l'outil choisi pour enregistrer les événements pertinents (connexion, déconnexion, messages, etc.).
- **Stockage:** Définisser un emplacement de stockage adapté pour les logs (local, distant, cloud).

**3. Mise en place de la Surveillance**
- **Outils de monitoring:** Utilisation de  `Prometheus` et `Grafana`pour surveiller l'état du serveur, la consommation des ressources, et les éventuelles alertes.

**4.Installation des Clients**
- **Disponibilité multiplateforme:** S'assurer que les clients Mumble sont disponibles pour tous les systèmes d'exploitation que vous souhaitez supporter (Windows, macOS, Linux).
- **Configuration des clients:** Configurer les clients pour se connecter au serveur avec les paramètres corrects.

**5. Tests et Optimisation**
- **Tests de connexion:** Vérifier que les clients peuvent se connecter au serveur et communiquer entre eux.
- **Tests de qualité:** Évaluer la qualité de l'audio et de la vidéo.
- **Optimisation des performances:** Ajuster les paramètres du serveur et des clients pour optimiser les performances en fonction de votre configuration matérielle et du nombre d'utilisateurs.

**6. Considérations Supplémentaires**
- **Documentation:** Créer une documentation complète pour faciliter la gestion et la maintenance du serveur.
 


# **Tutoriel : Installation de Mumble sur une VM VMware**
---
**1. Création de la machine virtuelle:**
- **Ouvrir VMware Workstation:** Lancez l'application VMware Workstation.
- **Créer une nouvelle machine virtuelle:** Suivez les étapes de création d'une nouvelle machine virtuelle, en sélectionnant l'image ISO de Debian 12.
- **Configurer la machine:** Attribuez suffisamment de ressources (CPU, RAM, disque dur) à votre machine virtuelle pour assurer un fonctionnement fluide de Mumble.

**2. Installation de Debian 12:**
- **Booter sur l'ISO:** Démarrez la machine virtuelle sur l'image ISO de Debian 12.
- **Installation:** Suivez les instructions d'installation de Debian, en choisissant les options appropriées (partitionnement, mot de passe root, etc.).

**3. Installation de Mumble:**
- **Mise à jour des paquets:** Une fois l'installation de Debian terminée, mettez à jour les paquets :
    
    Bash
    
    `sudo apt update && sudo apt upgrade`
    
- **Installation de Mumble:** Installez le serveur Mumble :
    
    Bash
    
    `sudo apt install mumble-server`
    
- **Configuration initiale:** Exécutez la configuration initiale :
Suivez les instructions pour configurer le mot de passe administrateur, le démarrage automatique, etc.
    
    Bash
    
    `sudo dpkg-reconfigure mumble-server`

**4. Configuration avancée (optionnel):**
- **Édition du fichier de configuration:** Pour des options de configuration plus fines, modifiez le fichier `/etc/mumble-server.ini`.
- **Sécurité:** Configurez un pare-feu pour autoriser uniquement les connexions entrantes sur le port utilisé par Mumble (par défaut 64738).
- **Journalisation:** Configurez la journalisation pour suivre l'activité du serveur.

# **Tutoriel : Installation de Mumble sur une VM Proxmox**
---
**1. Création de la machine virtuelle:**
- **Ouvrir Proxmox:** Connectez-vous à votre interface Proxmox.
- **Créer une nouvelle VM:** Suivez les instructions pour créer une nouvelle machine virtuelle, en sélectionnant l'image ISO de Debian 12.
- **Configurer la VM:** Attribuez les ressources nécessaires à votre VM.

**2. Installation de Debian 12:**
* **Booter sur l'ISO:** Démarrez la machine virtuelle et suivez les instructions d'installation de Debian, comme pour VMware.
* **Installation:** Suivez les instructions d'installation de Debian, en choisissant les options appropriées (partitionnement, mot de passe root, etc.).

**3. Installation de Mumble:**
- **Connexion en SSH:** Connectez-vous à votre VM via SSH.
- **Installation et configuration:** Suivez les mêmes étapes que pour l'installation sur VMware.

**Note:** Proxmox offre souvent des interfaces web pour gérer les machines virtuelles et effectuer certaines tâches, ce qui peut simplifier certaines étapes.

# **Tutoriel : Installation de Mumble sur une VM Virtual Box**
---
**1. Création de la machine virtuelle:**
- **Ouvrir Virtual Box:** Connectez-vous à votre interface Virtual Box.
- **Créer une nouvelle VM:** Suivez les instructions pour créer une nouvelle machine virtuelle, en sélectionnant l'image ISO de Debian 12.
- **Configurer la VM:** Attribuez les ressources nécessaires à votre VM.

**2. Installation de Debian 12:**
* **Booter sur l'ISO:** Démarrez la machine virtuelle et suivez les instructions d'installation de Debian, comme pour VMware et Proxmox.
* **Installation:** Suivez les instructions d'installation de Debian, en choisissant les options appropriées (partitionnement, mot de passe root, etc.).

**3. Installation de Mumble:**
- **Connexion en SSH:** Connectez-vous à votre VM via SSH.
- **Installation et configuration:** Suivez les mêmes étapes que pour l'installation sur VMware.

## **Points à considérer pour les deux environnements :**
- **Sécurité:** Assurez-vous que votre serveur Mumble est sécurisé en utilisant des mots de passe forts, en configurant un pare-feu et en mettant à jour régulièrement les logiciels.
- **Performances:** Ajustez les ressources allouées à votre machine virtuelle en fonction du nombre d'utilisateurs attendus.
- **Disponibilité:** Envisagez de mettre en place une redondance pour assurer la disponibilité de votre serveur.
- **Journalisation:** Configurez la journalisation pour pouvoir analyser les événements et dépanner en cas de problème.

### **Pour une configuration plus avancée, vous pouvez explorer les options suivantes:**

- **Plugins Mumble:** Mumble propose de nombreux plugins pour étendre ses fonctionnalités.
- **Intégration avec d'autres services:** Vous pouvez intégrer Mumble avec d'autres services comme Discord ou Slack.
- **Automatisation:** Utilisez des outils comme Ansible ou Puppet pour automatiser l'installation et la configuration de votre serveur.




Configuration d'un serveur de communication vocale sécurisé avec Mumble
Ce guide détaille la configuration d’un serveur Mumble sur Debian 12 pour des communications vocales sécurisées entre clients.

## 1. Introduction à Mumble
Mumble est une solution open-source pour les communications vocales en temps réel, populaire dans les environnements nécessitant faible latence et sécurité :
* Serveur : Murmur (composant serveur de Mumble).
* Client : Disponible pour Windows, macOS, Linux, Android, et iOS.

## 2. Prérequis
Serveur Debian 12 :
* Une instance Debian 12 avec accès root ou utilisateur sudo.
* Une connexion internet stable.
* Un certificat SSL (auto-signé ou fourni par Let's Encrypt).
* 
Client :
* Ordinateurs ou appareils équipés d’un client Mumble.
## 3. Installation et configuration du serveur Mumble
**Étape 1 :** Mettre à jour le serveur
`sudo apt update && sudo apt upgrade -y`

**Étape 2 :** Installer le serveur Murmur
`sudo apt install mumble-server -y`

**Étape 3 :** Configurer Murmur
1. Ouvrir le fichier de configuration :
`sudo nano /etc/mumble-server.ini`

2. Configurer les paramètres principaux :
* Mot de passe administrateur : Recherchez serverpassword et définissez un mot de passe pour sécuriser l'accès :
`serverpassword=VotreMotDePasse`
* Certificat SSL : Si vous utilisez un certificat SSL personnalisé, spécifiez le chemin :
`sslCert=/chemin/vers/certificat.pem
sslKey=/chemin/vers/cle.pem`
* Limiter le nombre de connexions (optionnel) :
`users=50`
Cela limite les connexions simultanées à 50 utilisateurs.
3. Sauvegarder et quitter :
Appuyez sur CTRL + O, puis CTRL + X.

**Étape 4 :** Démarrer et activer le serveur
`sudo systemctl start mumble-server`
`sudo systemctl enable mumble-server`

**Étape 5 :** Vérifier le statut du serveur
`sudo systemctl status mumble-server`
4. Configurer le pare-feu
Pour permettre aux clients de se connecter, ouvrez le port par défaut (64738) :
`sudo ufw allow 64738/tcp`
`sudo ufw allow 64738/udp`
`sudo ufw reload`
