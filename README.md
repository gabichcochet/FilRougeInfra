# FilRougeInfra

Projet scolaire Ynov Bachelor 2 combinant une solution INFRA multi-site et une application DEV immobilière.

## Vue d'ensemble

Ce dépôt contient deux volets principaux :

1. **Infrastructure** (`Contexte.md` et `Infra/README.md`) :
   - Simulation d’une infrastructure multi-site siège/agence.
   - 2 pare-feux pfSense en site-à-site VPN IPsec.
   - 1 contrôleur de domaine Windows Server Core `DC01`.
   - 1 serveur Windows Server GUI `WEB01` avec IIS, SQL Server Express, partages de fichiers et sauvegardes.
   - 2 postes clients Windows 11 : `CLIENT-SIEGE` et `CLIENT-AGENCE`.
   - Services AD, DNS, DHCP, GPO, droits NTFS, routage, et accès inter-sites.

2. **Application web** (`Dev/fil-rouge`) :
   - Plateforme immobilière `Loge-Moi` développée en PHP / MySQL.
   - Fonctionnalités de publication, recherche, comparaison, favoris, messagerie, rendez-vous et back-office.
   - Architecture orientée objet avec autoload PSR-4 maison, couche `src/`, et module d’analyse Python.

## Contenu du dépôt

- `Contexte.md` : documentation principale de l’infrastructure.
- `Infra/README.md` : documentation détaillée de l’infrastructure avec architecture, IP, VPN, DC01, WEB01, GPO, pfSense et vérifications.
- `Dev/fil-rouge/README.md` : documentation de l’application Web `Loge-Moi`.
- `Dev/fil-rouge/` : code source PHP de l’application immobilière.
- `Dev/fil-rouge/sql/` : scripts SQL de création de base, migrations et jeux de données de démonstration.

## Partie INFRA

La solution INFRA couvre :

- Topologie multi-site simulateur VirtualBox.
- `pfSense-SIEGE` avec WAN DHCP et LAN `192.168.1.1/24`.
- `pfSense-AGENCE` avec LAN `192.168.2.1/24`.
- VPN IPsec site-à-site entre `192.168.1.0/24` et `192.168.2.0/24`.
- Contrôleur AD et DNS : domaine `ymmo.local`, OU utilisateurs, OU ordinateurs, groupes de sécurité.
- `WEB01` : site IIS `web.ymmo.local`, serveur SQL, partages réseau.
- Clients Windows 11 intégrés au domaine, avec DNS sur `DC01` et mappage de lecteur réseau via GPO.
- DHCP géré par pfSense, routage et règles firewall pour limiter le trafic intersite aux services nécessaires.

## Partie DEV

L’application `Loge-Moi` inclut :

- Front-office : recherche de biens, galerie photos, comparateur, favoris, agences, prises de rendez-vous.
- Espace agent : gestion des biens, agenda, création/rejoindre agence, statistiques.
- Back-office admin : validation des candidatures agents et gestion des utilisateurs.
- Module messagerie interne avec rafraîchissement AJAX.
- Module Python d’analyse de données pour les agents.

## Démarrage rapide

### Infrastructure

Lire d’abord :
- `Contexte.md`
- `Infra/README.md`

Ces documents expliquent la topologie, les adresses IP, la configuration des pare-feux pfSense, la configuration de `DC01` et `WEB01`, les GPO et les contrôles.

### Application web

Lire :
- `Dev/fil-rouge/README.md`

Le dossier `Dev/fil-rouge` contient l’application PHP, les scripts SQL, les pages web et les composants backend.

## Notes importantes

- Le projet INFRA est principalement documentaire et de configuration ; il ne contient pas de code de déploiement automatique.
- Le projet DEV se déploie dans un environnement XAMPP / Apache / PHP 8+ avec MySQL/MariaDB.
- Le module d’analyse Python du projet DEV est optionnel et nécessite Python 3.9+.

## Structure recommandée

- `Contexte.md` et `Infra/README.md` pour la documentation INFRA.
- `Dev/fil-rouge/` pour l’application Web.

---

Pour plus de détails, consultez les documents internes : `Contexte.md`, `Infra/README.md` et `Dev/fil-rouge/README.md`.
