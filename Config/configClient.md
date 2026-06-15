📘 README — CLIENT01
Rôle : Poste client du domaine YMMO.LOCAL  
OS : Windows Server 2025 avec interface graphique (GUI)  
Nom de machine : CLIENT01  
Adresse IP : 10.0.0.11 (exemple)  
Domaine : YMMO.LOCAL

1. 🎯 Objectif de la machine
CLIENT01 sert de poste client pour :

tester l’intégration au domaine

vérifier les services AD DS (DNS, DHCP, GPO)

démontrer l’application des stratégies de groupe

simuler un poste utilisateur du Siège ou d’une Agence

valider la communication avec le routeur et DC01

2. 🖥️ Configuration réseau
2.1. Paramètres IP
powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.0.11 -PrefixLength 24 -DefaultGateway 10.0.0.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.10
IP : 10.0.0.11

Masque : /24

Passerelle : 10.0.0.1

DNS : 10.0.0.10 (DC01)

3. 🏢 Intégration au domaine
3.1. Renommage de la machine
powershell
Rename-Computer -NewName "CLIENT01" -Restart
3.2. Joindre le domaine YMMO.LOCAL
powershell
Add-Computer -DomainName "ymmo.local" -Credential ymmo\administrateur -Restart
4. 🔐 Vérifications post‑intégration
4.1. Vérifier l’identité
powershell
whoami
Doit afficher :

Code
ymmo\utilisateur
4.2. Vérifier la résolution DNS
powershell
ping DC01
4.3. Vérifier l’accès aux partages SYSVOL / NETLOGON
powershell
\\DC01\SYSVOL
\\DC01\NETLOGON
5. 🖼️ Application des GPO (exemple : fond d’écran)
5.1. Forcer l’application
powershell
gpupdate /force
5.2. Vérifier les GPO appliquées
powershell
gpresult /r
5.3. Vérifier l’accès au fond d’écran
powershell
\\DC01\Wallpapers\images.jpg
6. 🔧 Outils RSAT (optionnel mais recommandé)
Installation de la console GPMC pour gérer les GPO depuis CLIENT01 :

powershell
Add-WindowsFeature GPMC
7. 🔍 Tests fonctionnels
7.1. Test de connexion au domaine
Connexion avec un utilisateur du Siège ou d’une Agence

Vérification du profil utilisateur

Vérification des GPO appliquées

7.2. Test réseau
powershell
Test-NetConnection -ComputerName DC01 -Port 445
Test-NetConnection -ComputerName DC01 -Port 53
7.3. Test d’accès aux ressources
Accès aux partages

Accès Internet (si configuré via routeur)

8. 🧹 Maintenance & bonnes pratiques
Toujours utiliser un compte utilisateur normal pour tester les GPO

Ne pas utiliser le compte Administrateur du domaine pour les tests utilisateur

Redémarrer après toute modification majeure (GPO, réseau, domaine)

Vérifier régulièrement la synchronisation avec DC01

9. ✔ Résumé rapide
CLIENT01 est un poste client Windows Server GUI :

intégré au domaine YMMO.LOCAL

configuré avec DNS → DC01

utilisé pour tester GPO, DHCP, DNS, authentification

capable d’accéder aux partages SYSVOL / NETLOGON

prêt pour les démonstrations INFRA