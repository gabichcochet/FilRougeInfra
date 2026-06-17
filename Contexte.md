# Documentation INFRA — Architecture Siège + Agence

## Contexte du projet
Ce document décrit la solution d'infrastructure pour le projet INFRA de Ynov Bachelor 2.
Il présente l'architecture, les choix de configuration et les vérifications nécessaires pour un siège et une agence reliés par VPN IPsec.

## Objectif
Mettre en place une infrastructure multi-site simulée comprenant :
- 1 siège à Aix-en-Provence
- 1 agence site secondaire
- VPN IPsec site-à-site

## Composants clés
- 2 VM pfSense (SIEGE + AGENCE)
- 1 VM Windows Server Core DC01 (AD, DNS)
- 1 VM Windows Server GUI WEB01 (IIS, SQL, fichiers, sauvegardes)
- 2 VM clients Windows 11 (CLIENT-SIEGE + CLIENT-AGENCE)

## Services gérés
- Active Directory
- DNS
- DHCP
- GPO (mappage de lecteur réseau)
- Partages réseau et droits NTFS
- Routage inter-sites

## Sommaire
1. Architecture globale
2. Plan d'adressage IP
3. VPN IPsec
4. Configuration DC01
5. Configuration WEB01
6. GPO et partages
7. pfSense Siège
8. Clients Windows
9. Vérifications

🧱 2. Architecture globale (Version complète et correcte)

🧱 2. Architecture globale (Version complète et correcte)

```
                        INTERNET (NAT VirtualBox)
                                |
                =================================================
                                |
                    +-----------------------+
                    |     pfSense SIEGE     |
                    +-----------------------+
                       WAN: 10.0.2.5/15
                       LAN: 192.168.1.1/24
                                |
                        +----------------+
                        |   LAN-SIEGE    |
                        +----------------+
                         |       |       |
                 +-------------+ | +--------------+
                 |    DC01     | | | CLIENT-SIEGE |
                 +-------------+ | +--------------+
                 192.168.1.101   |   DHCP (192.168.1.0)
                                 |
                         +---------------+
                         |    WEB01      |
                         +---------------+
                         192.168.1.30
                         IIS / SQL / Fichiers

                        <=== VPN IPsec ===>

                    +-----------------------+
                    |    pfSense AGENCE     |
                    +-----------------------+
                       WAN: 10.0.2.4/15
                       LAN: 192.168.2.1/24
                                |
                        +----------------+
                        |  LAN-AGENCE    |
                        +----------------+
                         |         
                 +---------------+  
                 | CLIENT-AGENCE |  
                 +---------------+   
                 DHCP (192.168.2.200)
```

🧬 3. Plan d’adressage IP

🟩 SIEGE
Équipement	IP	Rôle
pfSense‑SIEGE	192.168.1.1	Passerelle
DC01	192.168.1.101	AD / DNS
WEB01	192.168.1.30	Web / DB / Fichiers
CLIENT‑SIEGE	DHCP	Client


DHCP pfSense :
192.168.1.100 → 192.168.1.200

🟦 AGENCE
Équipement	IP	Rôle
pfSense‑AGENCE	192.168.2.1	Passerelle
CLIENT‑AGENCE	DHCP	Client


DHCP pfSense :
192.168.2.100 → 192.168.2.200

👉 Aucune machine serveur dans l’agence (conforme au PDF).

🔐 4. VPN IPsec Site‑à‑Site
Phase 1
IKEv2

AES256

SHA256

DH Group 14

Phase 2
AES256

SHA256

PFS Group 14

Réseaux autorisés
192.168.1.0/24 ↔ 192.168.2.0/24

🖥️ 5. DC01 — Configuration complète
✔ Rôles
AD DS

DNS

SYSVOL

Partages réseau

GPO (Drive Map Direction)

✔ IP
192.168.1.101(DHCP)

Passerelle : 192.168.1.1

DNS : 127.0.0.1

✔ Active Directory

```
Domaine : ymmo.local

OU :


ymmo.local
 ├── Utilisateurs
 │     ├── Direction
 │     ├── Commercial
 │     ├── CommunicationMarketing
 │     ├── AdministratifRHJuridique
 │     └── ITSupport
 └── Ordinateurs
       ├── CLIENT-SIEGE
       └── CLIENT-AGENCE
✔ Groupes de sécurité
GG‑Direction

GG‑Commercial

GG‑CommunicationMarketing

GG‑AdministratifRHJuridique

GG‑ITSupport

✔ Partages réseau
Code
D:\Partages\
 ├── Direction
 ├── Commercial
 ├── CommunicationMarketing
 ├── AdministratifRHJuridique
 └── ITSupport
```


🟦 6. WEB01 — Configuration complète (SIEGE)
✔ Rôles
Serveur Web (IIS)

Serveur Base de données (SQL Server Express)

Serveur de fichiers (si besoin)

Serveur de sauvegardes

✔ IP
192.168.1.30

Passerelle : 192.168.1.1

DNS : 192.168.1.101 (DC01)

✔ Configuration IIS
Site : YmmoSite

Chemin : C:\Sites\Ymmo

Host header : web.ymmo.local

Bindings : HTTP 80

✔ DNS
Entrée A :

web.ymmo.local → 192.168.1.30

🖥️ 7. Clients Windows
CLIENT‑SIEGE
IP DHCP

DNS = DC01

Membre du domaine

GPO appliquées

Lecteur H: monté automatiquement

CLIENT‑AGENCE
IP DHCP

DNS = DC01 via VPN

Membre du domaine

Accès aux partages via VPN

🟩 8. GPO — Version finale

✔ GPO‑Mappage‑Direction (fonctionnelle)

GPO liée à l’OU : Direction
Filtrage de sécurité : GG‑Direction
Objet de stratégie : mappage du lecteur réseau H: vers \\DC01\Direction

Répartition des GPO prévue :
- Utilisateurs de l’OU Direction : GPO‑Mappage‑Direction
- Utilisateurs de l’OU Commercial : GPO‑Mappage‑Commercial (même logique)
- Utilisateurs de l’OU CommunicationMarketing : GPO‑Mappage‑CommunicationMarketing
- Utilisateurs de l’OU AdministratifRHJuridique : GPO‑Mappage‑AdministratifRHJuridique
- Utilisateurs de l’OU ITSupport : GPO‑Mappage‑ITSupport

Chaque GPO est appliquée via une liaison à l’OU utilisateur et un filtrage sur le groupe de sécurité correspondant, ce qui garantit une répartition claire et une application ciblée des stratégies.

Drive Map H:
Chemin : \\DC01\Direction
Filtrage : GG‑Direction
Lien : OU Direction

XML dans SYSVOL (créé manuellement)

Résultat sur CLIENT‑SIEGE :

Code
H    FileSystem    \\DC01\Direction
🟦 9. Services critiques (tous OK)
AD DS

DNS

Netlogon

KDC

DFSR

DHCP (pfSense)

VPN IPsec

IIS (WEB01)

SQL Server (WEB01)



### config DC01 

🔧 Vérification
powershell
ipconfig /all
🟦 3. Installation du rôle AD DS + DNS
🔧 Installer AD DS
powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
🔧 Promouvoir en contrôleur de domaine
powershell
Install-ADDSForest -DomainName "ymmo.local" -DomainNetbiosName "YMMO" -InstallDns -Force
Le serveur redémarre automatiquement.

🟩 4. Vérifications AD / DNS


🔧 Vérifier le domaine

```
powershell
Get-ADDomain
```

🔧 Vérifier la forêt

```
powershell
Get-ADForest
```

🔧 Vérifier DNS

```
powershell
Get-DnsServerResourceRecord -ZoneName "ymmo.local"
```

🔧 Vérifier SYSVOL

```
powershell
net share
```

Et on voit :

```
SYSVOL   C:\Windows\SYSVOL\sysvol
NETLOGON C:\Windows\SYSVOL\sysvol\ymmo.local\SCRIPTS
```

🟦 5. Structure Active Directory (OU + Groupes)
🔧 Créer les OU

```
powershell
New-ADOrganizationalUnit -Name "Utilisateurs" -Path "DC=ymmo,DC=local"
New-ADOrganizationalUnit -Name "Ordinateurs" -Path "DC=ymmo,DC=local"

New-ADOrganizationalUnit -Name "Direction" -Path "OU=Utilisateurs,DC=ymmo,DC=local"
New-ADOrganizationalUnit -Name "Commercial" -Path "OU=Utilisateurs,DC=ymmo,DC=local"
New-ADOrganizationalUnit -Name "CommunicationMarketing" -Path "OU=Utilisateurs,DC=ymmo,DC=local"
New-ADOrganizationalUnit -Name "AdministratifRHJuridique" -Path "OU=Utilisateurs,DC=ymmo,DC=local"
New-ADOrganizationalUnit -Name "ITSupport" -Path "OU=Utilisateurs,DC=ymmo,DC=local"
```

🔧 Créer les groupes de sécurité

```
powershell
New-ADGroup -Name "GG-Direction" -GroupScope Global -GroupCategory Security -Path "OU=Direction,OU=Utilisateurs,DC=ymmo,DC=local"
New-ADGroup -Name "GG-Commercial" -GroupScope Global -GroupCategory Security -Path "OU=Commercial,OU=Utilisateurs,DC=ymmo,DC=local"
New-ADGroup -Name "GG-CommunicationMarketing" -GroupScope Global -GroupCategory Security -Path "OU=CommunicationMarketing,OU=Utilisateurs,DC=ymmo,DC=local"
New-ADGroup -Name "GG-AdministratifRHJuridique" -GroupScope Global -GroupCategory Security -Path "OU=AdministratifRHJuridique,OU=Utilisateurs,DC=ymmo,DC=local"
New-ADGroup -Name "GG-ITSupport" -GroupScope Global -GroupCategory Security -Path "OU=ITSupport,OU=Utilisateurs,DC=ymmo,DC=local"
```

🟩 6. Dossiers partagés + droits NTFS
🔧 Créer l’arborescence

```
powershell
New-Item -ItemType Directory -Path "D:\Partages\Direction" -Force
New-Item -ItemType Directory -Path "D:\Partages\Commercial" -Force
New-Item -ItemType Directory -Path "D:\Partages\CommunicationMarketing" -Force
New-Item -ItemType Directory -Path "D:\Partages\AdministratifRHJuridique" -Force
New-Item -ItemType Directory -Path "D:\Partages\ITSupport" -Force
```

🔧 Partager les dossiers

```
powershell
New-SmbShare -Name "Direction" -Path "D:\Partages\Direction" -FullAccess "GG-Direction"
New-SmbShare -Name "Commercial" -Path "D:\Partages\Commercial" -FullAccess "GG-Commercial"
New-SmbShare -Name "CommunicationMarketing" -Path "D:\Partages\CommunicationMarketing" -FullAccess "GG-CommunicationMarketing"
New-SmbShare -Name "AdministratifRHJuridique" -Path "D:\Partages\AdministratifRHJuridique" -FullAccess "GG-AdministratifRHJuridique"
New-SmbShare -Name "ITSupport" -Path "D:\Partages\ITSupport" -FullAccess "GG-ITSupport"
```

🔧 Appliquer les droits NTFS (exemple Direction)

```
powershell
icacls "D:\Partages\Direction" /inheritance:r
icacls "D:\Partages\Direction" /grant "GG-Direction:(OI)(CI)F"
icacls "D:\Partages\Direction" /grant "GG-Commercial:(OI)(CI)R"
icacls "D:\Partages\Direction" /grant "GG-CommunicationMarketing:(OI)(CI)R"
icacls "D:\Partages\Direction" /grant "GG-AdministratifRHJuridique:(OI)(CI)R"
icacls "D:\Partages\Direction" /grant "GG-ITSupport:(OI)(CI)R"
```

🟦 7. GPO — Mappage du lecteur H: (Drive Map)
🔧 Créer le dossier GPP dans SYSVOL

```
powershell
$GPOGuid = "{8A03BF20-2F30-4D8A-A286-15A108ED480D}"
$DrivesPath = "\\DC01\SYSVOL\ymmo.local\Policies\$GPOGuid\User\Preferences\Drives"
New-Item -ItemType Directory -Path $DrivesPath -Force
🔧 Créer le fichier Drives.xml`
@"
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{79F92669-4224-476c-9C5C-6EFB4D87DF4A}">
  <Drive clsid="{F5B39A0A-83A0-4a5b-8C8F-9C2A5A5C9E30}" name="H" status="H" image="0" changed="2026-06-16 00:00:00" uid="{11111111-2222-3333-4444-555555555555}" userContext="1" bypassErrors="1">
    <Properties action="C" thisDrive="H" useLetter="1" letter="H" path="\\DC01\Direction" persistent="1" />
    <Filters />
  </Drive>
</Drives>
"@ | Set-Content "$DrivesPath\Drives.xml" -Encoding UTF8
```

🔧 Vérifier
`
```
powershell
Get-ChildItem $DrivesPath
```

🟩 8. Vérifications finales
🔧 Vérifier Kerberos

```
powershell
klist
```

🔧 Vérifier les services

```
powershell
Get-Service adws,dns,dfsr,netlogon,kdc
```

🔧 Vérifier la réplication SYSVOL

```
powershell
dcdiag /test:sysvolcheck
```

🔧 Vérifier les GPO

```
powershell
gpresult /r
```

### config pfSense Siège

🔥 pfSense-SIEGE — Configuration finale
Rôle : pare-feu, NAT, DHCP, passerelle LAN et endpoint VPN principal

#### 1. Informations générales

- Nom : pfSense-SIEGE
- Version : pfSense CE 2.7.x
- WAN : DHCP (VirtualBox NAT)
- LAN : 192.168.1.1/24
- Accès Web : https://192.168.1.1
- Identifiants : admin / pfsense

#### 2. Interfaces


Menu : Interfaces > Assignments

| Interface | NIC | IP | Rôle |
|-----------|-----|----|------|
| WAN | em0 | DHCP (10.0.2.15/24) | Sortie Internet |
| LAN | em1 | 192.168.1.1/24 | Réseau interne |

Menu : Interfaces > LAN
- IPv4 Configuration Type : Static IPv4
- IPv4 Address : 192.168.1.1 / 24
- IPv6 : None
- DHCP Relay : Disabled


#### 3. DHCP Server (LAN)


Menu : Services > DHCP Server > LAN

| Paramètre | Valeur |
|-----------|--------|
| DHCP Server | Enabled |
| Range | 192.168.1.20 → 192.168.1.50 |
| DNS Server | 192.168.1.10 (DC01) |
| Gateway | 192.168.1.1 |
| Domain Name | ymmo.local |

👉 Cette configuration est correcte pour le siège.


#### 4. NAT & Firewall Rules


##### NAT


Menu : Firewall > NAT > Outbound
- Mode : Automatic Outbound NAT
- Règles générées automatiquement pour le WAN


##### Firewall Rules (LAN)


Menu : Firewall > Rules > LAN
- Autoriser LAN net → any : permet aux postes du siège d’accéder à Internet, à DC01 et à WEB01.
- Autoriser LAN net → 192.168.2.0/24 : nécessaire pour le VPN avec l’agence.
- Autoriser 192.168.2.0/24 → 192.168.1.10 / 192.168.1.30 sur les ports nécessaires :
  - DNS 53
  - Kerberos 88
  - LDAP 389
  - SMB 445
  - RPC 135
  - Global Catalog 3268
  - HTTP 80
- Bloquer tout le reste entre 192.168.2.0/24 et 192.168.1.0/24 par défaut pour limiter le trafic intersite aux services nécessaires.

##### Firewall Rules (WAN)


Menu : Firewall > Rules > WAN
- Action : Block
- Source : any
- Destination : any

#### 5. DNS Resolver
Menu : Services > DNS Resolver

```
| Paramètre | Valeur |
|-----------|--------|
| DNS Resolver | Enabled |
| Register DHCP leases | Enabled |
| Register DHCP static mappings | Enabled |
| DNSSEC | Enabled |
```

#### 6. Accès Web pfSense
- URL : https://192.168.1.1
- Certificat auto-signé : avertissement normal
- Identifiants : admin / pfsense

#### 7. Configuration pour Active Directory

```
- Trafic vers DC01 autorisé : OK
- Ports nécessaires autorisés (via la règle LAN) :
  - 53 DNS
  - 88 Kerberos
  - 389 LDAP
  - 445 SMB
  - 135 RPC
  - 464 Kerberos password
  - 3268 Global Catalog
```

#### 8. Préparation VPN IPsec (SIEGE)
pfSense-SIEGE est prêt pour le VPN. Il reste à configurer la partie AGENCE.

- Phase 1 : IKEv2, AES256, SHA256, DH Group 14
- Phase 2 : AES256, SHA256, PFS Group 14
- Réseaux autorisés : Local 192.168.1.0/24, Remote 192.168.2.0/24


### config Utilisateur Siège

🖥️ CLIENT‑SIEGE — Configuration complète (Windows 11 Pro)
Rôle : Poste client du siège, intégré au domaine, utilisé pour tester AD, DNS, GPO, mappage réseau.

🟦 1. Informations générales
Élément	Valeur

Nom du poste	CLIENT‑SIEGE
OS	Windows 11 Pro
Domaine	ymmo.local
Réseau	LAN‑SIEGE
Adresse IP	DHCP (192.168.1.x)
Passerelle	192.168.1.1 (pfSense‑SIEGE)
DNS	192.168.1.10 (DC01)


🟩 2. Configuration réseau
✔ Adresse IP (DHCP)
Commande :

powershell

```
ipconfig /all
```

Résultat attendu :

```
IPv4 Address. . . . . . . . . . . : 192.168.1.103 (DHCP)
Subnet Mask . . . . . . . . . . . : 255.255.255.0
Default Gateway . . . . . . . . . : 192.168.1.1
DNS Servers . . . . . . . . . . . : 192.168.1.10
👉 DNS = DC01, obligatoire pour AD.`
```

🟦 3. Intégration au domaine
✔ Commande PowerShell utilisée

```
powershell
Add-Computer -DomainName "ymmo.local" -Credential ymmo\Administrateur
Restart-Computer
```

✔ Vérification

```
powershell
whoami /fqdn`
```

Résultat :

```
client-siege.ymmo.local
```

🟩 4. Position dans Active Directory
CLIENT‑SIEGE est placé dans l’OU :

```
ymmo.local
 └── Ordinateurs
       └── CLIENT-SIEGE
```


🟦 5. Tests DNS / AD / Kerberos
✔ Résolution DNS`

```
powershell
nslookup dc01.ymmo.local
Résultat attendu :
```


```
Name: dc01.ymmo.local
Address: 192.168.1.10
```

✔ Résolution SRV AD

```
powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.ymmo.local
```

✔ Ticket Kerberos

```
powershell
klist
```

Résultat attendu :

```
krbtgt/ymmo.local
```


🟩 6. GPO appliquées
✔ Commande

```
powershell
gpresult /r
```

✔ GPO Machine
Default Domain Policy

(Éventuelles GPO de sécurité si ajoutées)

✔ GPO Utilisateur
Default Domain Policy

GPO‑Mappage‑Direction (si l’utilisateur appartient à GG‑Direction)

🟦 7. Mappage réseau automatique (Drive Map)
✔ Commande

```
powershell
Get-PSDrive
```

Résultat attendu :

```
H    FileSystem    \\DC01\Direction
```

👉 Preuve que la GPO Drive Map fonctionne.

✔ Vérification via net use

```
powershell
net use
```

Résultat :

```
H:  \\DC01\Direction
```

🟩 8. Tests de connectivité
✔ Ping DC01

```
powershell
ping 192.168.1.10
```

✔ Ping WEB01

```
powershell
ping 192.168.1.
```

✔ Accès au partage

```
powershell
Test-Path \\DC01\Direction
```

Résultat :

```
True
```

🟦 9. Commandes utiles exécutées sur CLIENT‑SIEGE

🔧 Vérifier l’IP

```
powershell
ipconfig /all
```

🔧 Vérifier l’appartenance au domaine

```
powershell
whoami /fqdn
```

🔧 Vérifier les GPO

```
powershell
gpresult /r
```

🔧 Vérifier le lecteur réseau

```
powershell
Get-PSDrive
```

🔧 Vérifier Kerberos

```
powershell
klist
```

