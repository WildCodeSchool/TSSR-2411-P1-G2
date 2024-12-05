# TSSR-2411-P1-G2

# Projet: Serveur de communication vocal 



![](Ressources/Icons_mumble.svg.png)


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
* Logiciels: "Prontail", "Loki", et "Grafana" pour surveiller et journaliser les sessions.

---

## **Membres du Groupe de Projet et Rôles**
- **Scrum Master**

Barbara, experte dans la rédaction, maitrise le Markdown, bilingue (on prévoit un déploiement en Italie), comprend aussi bien les enjeux que la technique.

- **Product Owner**

Saddem, très bon porte-parole grâce à sa voix qui impose le respect, apporte une grande aide aux développeurs avec sa connaissance approfondie de la programmation et du réseau.

- **Développeurs**

Alexandre (Flosh) , try-harder de la programmation, son repas préféré : une grosse assiette de configuration.ini saupoudrée de documentation.md.

Julien, navigue à vue entre ses vms sur Proxmox, surf sur les scripts et apprécie les pool d’adresse IP.

---

## **SPRINT 1**
- Comprendre le fonctionnent du logicielle
- Analyser l’effet des modifications du fichier de configuration
- Résoudre les 1er problèmes techniques
- Création des VM , installation des logicielles et configuration

## **SPRINT 2**
- Ajout de l’interface web de Mumble
- Ajout d’une solution de monitoring de logs
- Rédaction de la documentation claire et détaillée
- Optimisation, ajout de fonctionnalités et rédaction de la documentation

---

# **Prérequis Techniques**

## Matériel
* Ordinateur

## Logiciel
* Systèmes d'exploitation Debian 12.iso.
* Server Mumble.
* VMware, Virtual Box ou Proxmox.
* Un accès SSH au serveur Mumble une fois installé.

# **Étapes Détaillées du Projet**:
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
 



`sudo ufw allow 64738/udp`
`sudo ufw reload`
