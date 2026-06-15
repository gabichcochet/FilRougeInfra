📘 README / Documentation du Projet YMMO — Infrastructure SIEGE & AGENCE
🏢 1. Objectif du projet
Mettre en place une infrastructure réseau complète simulant deux sites d’entreprise :

SIEGE (site principal)

AGENCE (site secondaire)

Les deux sites doivent être reliés via un VPN site‑à‑site, permettant la communication entre les réseaux internes.

L’infrastructure doit inclure :

Pare‑feu / routeurs (pfSense)

Serveurs Windows (AD, DNS, DHCP, Web)

Clients Windows

Séparation des réseaux

Routage inter‑sites

Sécurité réseau

🧱 2. Architecture globale
🌐 Vue d’ensemble


```
                INTERNET (NAT VirtualBox)
                        |
                +----------------+
                |  pfSense SIEGE |
                +----------------+
                   WAN: 10.0.2.15
                   LAN: 192.168.1.1/24
                        |
                +----------------+
                |   LAN-SIEGE    |
                +----------------+
                 |            |
            +---------+   +---------+
            |  DC01   |   | CLIENT01|
            +---------+   +---------+
            192.168.1.10   DHCP

                        VPN IPsec
                =======================
                        |
                +----------------+
                | pfSense AGENCE |
                +----------------+
                   WAN: 10.0.2.x
                   LAN: 192.168.2.1/24
                        |
                +----------------+
                |  LAN-AGENCE    |
                +----------------+
                 |            |
            +---------+   +---------+
            | SRV-AG |   | CLIENT02 |
            +---------+   +---------+
```

🖥️ 3. Choix des machines
🔥 pfSense (x2)
Rôle :

Pare‑feu

NAT

DHCP (optionnel)

VPN site‑à‑site

Routage inter‑réseaux

Pourquoi pfSense ?

Gratuit

Professionnel

Très utilisé en entreprise

Parfait pour simuler des environnements multi‑sites

🧩 Windows Server 2022 (x2)
DC01 (SIEGE)
Active Directory

DNS

DHCP (si pas géré par pfSense)

Contrôleur principal du domaine

SRV‑AGENCE (AGENCE)
Serveur secondaire

Peut héberger un rôle (fichier, web, réplica AD, etc.)

🖥️ Windows 11 (x1)
Poste client

Permet de tester l’intégration au domaine

Vérifier les règles de pare‑feu

Tester le VPN inter‑sites

🧬 4. Plan d’adressage
🟩 Site SIEGE
Équipement	IP	Rôle
pfSense‑SIEGE (LAN)	192.168.1.1	Passerelle
DC01	192.168.1.10	AD / DNS
CLIENT01	DHCP	Client


Plage DHCP pfSense :
192.168.1.10 → 192.168.1.50

🟦 Site AGENCE
Équipement	IP	Rôle
pfSense‑AGENCE (LAN)	192.168.2.1	Passerelle
SRV‑AGENCE	192.168.2.10	Serveur
CLIENT02	DHCP	Client


Plage DHCP pfSense :
192.168.2.10 → 192.168.2.50

🔐 5. VPN Site‑à‑Site (IPsec)
Phase 1
IKEv2

AES256

SHA256

DH Group 14

Phase 2
AES256

SHA256

PFS Group 14

Réseaux :

192.168.1.0/24 ↔ 192.168.2.0/24

### 🧩 6. Configuration actuelle de pfSense‑SIEGE
Voici l’état réel de ton pfSense‑SIEGE (vu sur ton écran) :

🔥 Interfaces
Code
WAN (wan) -> em0 -> v4/DHCP4: 10.0.2.15/24
LAN (lan) -> em1 -> v4: 192.168.1.1/24
🟩 DHCP LAN
Activé

Plage : 192.168.1.10 → 192.168.1.50

🔐 Accès Web
URL : https://192.168.1.1 (192.168.1.1 in Bing)

Login : admin

Password : pfsense

🧱 Fonctionnement
pfSense‑SIEGE est stable

Interfaces correctement assignées

LAN opérationnel

DHCP opérationnel

Prêt pour le VPN