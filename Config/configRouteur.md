Configuration du Routeur (RRAS – Windows Server)
🎯 Rôle du routeur
Le routeur assure le routage entre :
 
Internet (NAT VirtualBox)

Le LAN du siège (10.0.0.0/24)

Le réseau Host‑Only (192.168.183.0/24) utilisé pour l’administration depuis le PC physique

Il fournit également :

NAT pour permettre aux machines du LAN d’accéder à Internet

Routage IP entre les différents réseaux

Passerelle par défaut pour le LAN (10.0.0.1)

🧩 1. Architecture réseau du routeur
Le routeur possède 3 interfaces réseau :

Interface	Rôle	Type VirtualBox	Adresse IP
WAN	Accès Internet	NAT	10.0.2.15/24
LAN	Réseau interne du siège	Réseau interne (LAN-SIEGE)	10.0.0.1/24
Host‑Only	Communication PC ↔ Routeur	Host‑Only	192.168.183.2/24


🔧 2. Configuration IP
WAN (NAT VirtualBox)
Code
Interface : WAN
IPv4      : 10.0.2.15
Masque    : 255.255.255.0
Passerelle: 10.0.2.2
DNS       : 8.8.8.8 / 1.1.1.1
DHCP      : Oui (VirtualBox NAT)
LAN (Réseau interne du siège)
Code
Interface : LAN
IPv4      : 10.0.0.1
Masque    : 255.255.255.0
Passerelle: (aucune)
DNS       : 10.0.0.10 (DC01)
Host‑Only (Administration)
Code
Interface : Ethernet
IPv4      : 192.168.183.2
Masque    : 255.255.255.0
Passerelle: (aucune)
🔥 3. Configuration RRAS (Routing and Remote Access)
RRAS est configuré en mode :

NAT

Routage LAN ↔ WAN

Routage Host‑Only ↔ LAN

Interfaces NAT
Commande utilisée :

Code
netsh routing ip nat add interface "WAN" full
netsh routing ip nat add interface "LAN" private
netsh routing ip nat add interface "Ethernet" private
Vérification
Code
netsh routing ip nat show interface
Résultat attendu :

WAN → full

LAN → private

Ethernet (Host‑Only) → private

🔐 4. Pare‑feu Windows
Pour permettre les tests ICMP (ping), une règle a été ajoutée :

Code
New-NetFirewallRule -DisplayName "Allow ICMPv4 All" -Protocol ICMPv4 -Direction Inbound -Action Allow -Profile Domain,Private,Public
🔁 5. Routage IP
Le routage IP est activé automatiquement par RRAS :

Code
Routage IP activé : Oui
🧪 6. Tests de validation
Depuis le PC physique :
Code
ping 192.168.183.2   → OK (routeur Host‑Only)
ping 10.0.0.1        → OK (routeur LAN)
ping 10.0.0.10       → OK (DC01)
Depuis DC01 :
Code
ping 10.0.0.1        → OK
ping 192.168.183.2   → OK
Depuis une VM du LAN :
Code
ping 10.0.0.1        → OK
ping 8.8.8.8         → OK (Internet via NAT)
🧭 7. Rôle du routeur dans l’infrastructure
Fournit la passerelle du LAN (10.0.0.1)

Permet l’accès Internet aux serveurs et clients

Assure la communication entre :

PC physique ↔ LAN

LAN ↔ Internet

Supporte DHCP, DNS, AD, GPO via DC01

Simule un routeur d’entreprise dans la maquette INFRA