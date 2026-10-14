**SAMBA 4**

**Active Directory Domain Controller**

**Guide d'installation et de configuration complet**

Deux VMs Ubuntu 24.04.4 LTS • Serveur AD DC • Client graphique joint au domaine

Samba 4 AD DC • Kerberos • DNS interne • realmd • SSSD • Nautilus

# **1\. Comprendre l'architecture**

Avant de taper la première commande, il est essentiel de comprendre pourquoi Active Directory fonctionne comme il le fait. Ce n'est pas juste un annuaire LDAP - c'est un ensemble de trois protocoles qui travaillent ensemble, et ignorer l'un d'eux est la source principale d'échecs lors de l'installation.

## **1.1 Les trois piliers d'un Active Directory**

Un domaine Active Directory repose sur trois services indissociables. Premièrement, le DNS : chaque machine du domaine se localise via des enregistrements DNS spéciaux appelés SRV records. Sans DNS fonctionnel, les clients ne peuvent pas trouver le contrôleur de domaine, point. C'est la cause numéro un des échecs de jonction de domaine.

Deuxièmement, Kerberos : c'est le protocole d'authentification d'AD. Quand un utilisateur tape son mot de passe, ce n'est pas le mot de passe lui-même qui circule sur le réseau - Kerberos échange des « tickets » cryptés avec une durée de vie limitée. Cela rend l'interception inutile car un ticket expiré est sans valeur.

Troisièmement, LDAP : c'est le protocole d'annuaire qui stocke et expose la liste des utilisateurs, groupes, ordinateurs et politiques. Quand SSSD cherche les informations d'un utilisateur, il parle LDAP. Mais l'authentification elle-même passe par Kerberos, pas LDAP.

## **1.2 Ce que fait Samba 4 AD DC**

Samba 4 en mode contrôleur de domaine (AD DC) implémente les trois protocoles ci-dessus simultanément dans un seul service. Il se comporte exactement comme un Windows Server 2008 R2 du point de vue des clients - un client Ubuntu, Windows 10 ou Windows 11 ne fait pas la différence.

La différence importante avec le guide précédent : on n'installe plus slapd séparément. Samba 4 AD DC inclut sa propre instance LDAP interne (Heimdal pour Kerberos, et un LDAP basé sur LDB). Il ne faut surtout pas installer slapd sur le même serveur - les deux rentreraient en conflit sur le port 389.

## **1.3 Schéma de l'architecture**

Le domaine s'appellera MONLABO.LOCAL dans ce guide. Le suffixe .local est adapté aux labs - en production on utilise un vrai nom de domaine enregistré. Toutes les références à MONLABO.LOCAL doivent être remplacées en cohérence si tu choisis un autre nom.

| **VM 1 - SERVEUR AD DC**<br><br>Ubuntu 24.04.4 Server (sans GUI)<br><br>IP fixe : 192.168.56.10<br><br>Hostname : dc1.monlabo.local<br><br>**Samba 4 AD DC**<br><br>DNS interne du domaine<br><br>Kerberos (KDC)<br><br>LDAP Active Directory<br><br>Partages Samba (SYSVOL, NETLOGON, Shared) | ⟷   | **VM 2 - CLIENT DOMAINE**<br><br>Ubuntu 24.04.4 Desktop (GNOME)<br><br>IP fixe : 192.168.56.20<br><br>Hostname : poste01.monlabo.local<br><br>**realmd + SSSD**<br><br>Jonction au domaine AD<br><br>Authentification Kerberos<br><br>Accès partages via Nautilus |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

**ℹ Note :** Les deux VMs doivent partager un réseau Host-Only dans VirtualBox (vboxnet0, sous-réseau 192.168.56.0/24). Chaque VM a aussi un adaptateur NAT pour accéder à internet pendant l'installation.

# **2\. Spécifications des VMs**

Ces spécifications sont calibrées pour Ubuntu 24.04. Le client Desktop nécessite plus de RAM que sur 22.04 car GNOME 46 (inclus dans 24.04) est plus gourmand que GNOME 42.

## **2.1 VM 1 - Serveur Active Directory (AD DC)**

| **Paramètre**          | **Valeur recommandée**         | **Remarque**                                                          |
| ---------------------- | ------------------------------ | --------------------------------------------------------------------- |
| Système d'exploitation | Ubuntu 24.04.4 LTS Server      | ISO : ubuntu-24.04.4-live-server-amd64.iso - sans interface graphique |
| RAM                    | 2 Go minimum / 4 Go recommandé | Samba AD DC + Kerberos + DNS consomme plus qu'un simple slapd         |
| CPU                    | 2 vCPUs                        | 1 vCPU suffira en lab mais ralentit la provision Kerberos             |
| Disque                 | 30 Go alloué dynamiquement     | La base LDAP d'AD et SYSVOL peuvent grossir avec le temps             |
| Adaptateur réseau 1    | NAT                            | Pour télécharger les paquets pendant l'installation                   |
| Adaptateur réseau 2    | Host-Only - vboxnet0           | Réseau interne VM↔VM - doit être en 192.168.56.0/24                   |
| Nom d'hôte (hostname)  | dc1                            | Le FQDN sera dc1.monlabo.local                                        |
| IP fixe (adaptateur 2) | 192.168.56.10/24               | Configurée après installation - critique pour le DNS AD               |
| Chipset VirtualBox     | PIIX3 ou ICH9                  | ICH9 donne parfois enp0s8 au lieu de enp0s3 - vérifier avec ip link   |

## **2.2 VM 2 - Client graphique joint au domaine**

| **Paramètre**          | **Valeur recommandée**             | **Remarque**                                             |
| ---------------------- | ---------------------------------- | -------------------------------------------------------- |
| Système d'exploitation | Ubuntu 24.04.4 LTS Desktop         | ISO : ubuntu-24.04.4-desktop-amd64.iso - inclut GNOME 46 |
| RAM                    | 4 Go minimum / 6 Go recommandé     | GNOME 46 sur 24.04 consomme ~1,5 Go au repos             |
| CPU                    | 2 vCPUs                            | Minimum pour une interface fluide                        |
| Disque                 | 30 Go alloué dynamiquement         | Prévoir de l'espace pour les profils utilisateurs LDAP   |
| Adaptateur réseau 1    | NAT                                | Pour télécharger les paquets                             |
| Adaptateur réseau 2    | Host-Only - vboxnet0               | Même réseau que le serveur                               |
| Mémoire vidéo          | 128 Mo minimum / 256 Mo recommandé | Augmenter si l'écran clignote ou si GNOME crashe         |
| Accélération 3D        | Activée (VBoxSVGA)                 | Améliore les performances GNOME 46                       |
| Nom d'hôte             | poste01                            | FQDN : poste01.monlabo.local                             |
| IP fixe (adaptateur 2) | 192.168.56.20/24                   | Doit pointer vers dc1 (192.168.56.10) comme DNS          |

# **3\. Préparation du serveur (VM 1)**

Cette section couvre l'installation d'Ubuntu Server 24.04, la configuration réseau, et la mise à jour du système avant l'installation de Samba. Ces étapes de préparation sont critiques - une IP ou un hostname mal configuré rendra l'AD DC inutilisable.

## **3.1 Installation d'Ubuntu Server 24.04**

Lors du démarrage sur l'ISO, l'installateur en mode texte (Subiquity) te guide étape par étape. Voici les choix importants à faire.

- Choisis « Ubuntu Server » (pas la version minimisée) pour inclure tous les utilitaires réseau.
- À l'étape réseau, configure uniquement l'adaptateur NAT (enp0s3) en DHCP pour l'instant - l'IP fixe viendra après.
- Définis le hostname comme dc1.
- Crée un utilisateur administrateur local (ex : srvadmin) - ce compte est distinct des utilisateurs AD.
- Active OpenSSH server si tu veux administrer le serveur depuis ton hôte (pratique pour copier-coller les commandes).
- Laisse le reste par défaut et termine l'installation.

## **3.2 Configuration de l'IP fixe sur l'interface Host-Only**

Sur Ubuntu 24.04 Server, Netplan est toujours le gestionnaire réseau par défaut, avec systemd-networkd comme backend. La configuration est identique à 22.04, mais il faut d'abord identifier le nom exact des interfaces.

### **Étape 1 - Identifier les interfaces réseau**

ip link show

\# Tu dois voir deux interfaces (hors lo) :

\# - enp0s3 (ou similaire) → adaptateur NAT, déjà configuré par DHCP

\# - enp0s8 (ou enp0s9) → adaptateur Host-Only, à configurer

### **Étape 2 - Créer le fichier Netplan pour l'interface Host-Only**

sudo nano /etc/netplan/99-hostonly.yaml

Contenu du fichier (attention : YAML utilise des espaces, jamais de tabulations) :

network:

version: 2

renderer: networkd

ethernets:

enp0s8: # Remplace par ton nom d'interface si différent

dhcp4: false

addresses:

\- 192.168.56.10/24

nameservers:

addresses: \[127.0.0.1\] # Le serveur est son propre DNS une fois AD configuré

sudo chmod 600 /etc/netplan/99-hostonly.yaml # Netplan exige des droits stricts

sudo netplan apply

ip addr show enp0s8 # Doit afficher 192.168.56.10/24

**⚠ Important :** La ligne 'nameservers: addresses: \[127.0.0.1\]' pointe vers le serveur lui-même. Cela ne fonctionnera qu'une fois Samba AD DC démarré et configuré comme DNS. Pour l'installation initiale, tu peux temporairement mettre 8.8.8.8 puis revenir à 127.0.0.1 après.

### **Étape 3 - Configurer le hostname et le fichier hosts**

C'est une étape critique. Le FQDN (Fully Qualified Domain Name) doit être résolvable localement avant même que le DNS AD soit actif, sinon la provision Samba échouera.

sudo hostnamectl set-hostname dc1.monlabo.local

\# Vérification :

hostname # Doit retourner : dc1.monlabo.local

hostname -s # Doit retourner : dc1

hostname -d # Doit retourner : monlabo.local

sudo nano /etc/hosts

\# Assure-toi que ces deux lignes sont présentes (et SEULEMENT ces deux pour 192.168.56.10) :

127.0.0.1 localhost

192.168.56.10 dc1.monlabo.local dc1

\# Supprimer toute autre ligne qui contiendrait dc1 ou monlabo

\# Test de résolution locale :

getent hosts dc1.monlabo.local # Doit retourner : 192.168.56.10

## **3.3 Mise à jour et installation des dépendances**

sudo apt update && sudo apt full-upgrade -y

\# Paquets nécessaires avant Samba :

sudo apt install -y acl attr build-essential libacl1-dev libattr1-dev \\

libblkid-dev libgnutls28-dev libreadline-dev python3-dev python3-dnspython \\

python3-gpg python3-markdown python3-packaging python3-subunit \\

xsltproc dnsutils krb5-user samba winbind libnss-winbind \\

libpam-winbind ntp ntpdate

\# Pendant l'installation de krb5-user, trois questions sont posées :

\# Realm Kerberos → MONLABO.LOCAL (en MAJUSCULES, obligatoire)

\# Serveur Kerberos → dc1.monlabo.local

\# Serveur admin → dc1.monlabo.local

**ℹ Note :** Le realm Kerberos DOIT être en majuscules (MONLABO.LOCAL). C'est une convention Kerberos stricte, pas une préférence. Un realm en minuscules causera des erreurs d'authentification cryptiques plus tard.

# **4\. Provisionnement de Samba en Active Directory DC**

La provision est l'étape qui initialise la base de données Active Directory. Elle crée le domaine, configure Kerberos, génère les zones DNS, et prépare SYSVOL (le partage utilisé pour distribuer les stratégies de groupe). C'est une opération à ne faire qu'une fois - recommencer signifie tout effacer et repartir de zéro.

## **4.1 Arrêter et désactiver les services qui entrent en conflit**

Samba AD DC intègre son propre serveur DNS et sa propre implémentation LDAP. Avant la provision, il faut s'assurer qu'aucun service existant ne tente d'occuper les mêmes ports.

\# Arrêter les services potentiellement conflictuels :

sudo systemctl stop smbd nmbd winbind

sudo systemctl disable smbd nmbd winbind

\# Sur Ubuntu 24.04, systemd-resolved écoute sur le port 53 par défaut.

\# Samba AD DC a besoin de ce port pour son DNS intégré - il faut donc libérer le port :

sudo systemctl disable --now systemd-resolved

\# Reconfigurer la résolution DNS temporairement (jusqu'au démarrage d'AD) :

sudo rm /etc/resolv.conf # C'est un lien symbolique vers systemd-resolved

echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf

echo 'nameserver 1.1.1.1' | sudo tee -a /etc/resolv.conf

**⚠ Important :** Désactiver systemd-resolved est une étape spécifique à Ubuntu 24.04 pour les serveurs AD. Ne pas faire cela = Samba ne peut pas démarrer son DNS interne = les clients ne trouvent jamais le contrôleur de domaine.

## **4.2 Supprimer la configuration Samba existante**

Ubuntu installe Samba avec un smb.conf par défaut. La commande de provision écrase ce fichier, mais il vaut mieux le supprimer manuellement pour éviter tout résidu.

sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.bak

\# Vider aussi les anciens fichiers de base de données Samba si présents :

sudo find /var/lib/samba -name '\*.ldb' -delete

sudo find /var/lib/samba -name '\*.tdb' -delete

## **4.3 Lancer la provision du domaine Active Directory**

La commande samba-tool domain provision crée l'intégralité du domaine AD. Chaque option est expliquée ci-dessous. Exécuter cette commande en une seule fois (les backslashes permettent de continuer sur la ligne suivante dans le terminal).

sudo samba-tool domain provision \\

\--use-rfc2307 \\

\--realm=MONLABO.LOCAL \\

\--domain=MONLABO \\

\--server-role=dc \\

\--dns-backend=SAMBA_INTERNAL \\

\--adminpass='VotreMotDePasseAdmin123!'

\# Explications :

\# --use-rfc2307 → Active les attributs POSIX (uidNumber, gidNumber)

\# Indispensable pour que les clients Linux lisent les UID/GID

\# --realm → Nom du realm Kerberos en MAJUSCULES

\# --domain → Nom NetBIOS du domaine (max 15 caractères, sans point)

\# --server-role=dc → Ce serveur est un contrôleur de domaine

\# --dns-backend=SAMBA_INTERNAL → Samba gère son propre DNS (recommandé pour les labs)

\# --adminpass → Mot de passe du compte Administrator AD

\# Doit contenir majuscules, minuscules, chiffres, symboles

**✔ Conseil :** Mémorise ou note bien le mot de passe Administrator AD - c'est le compte équivalent au 'root' du domaine. Il sert à créer tous les autres utilisateurs et à joindre les machines au domaine.

Si la provision réussit, tu verras un résumé avec les informations du domaine. Samba aura créé un nouveau smb.conf adapté à l'AD DC, ainsi que la structure SYSVOL dans /var/lib/samba/sysvol/.

## **4.4 Configurer Kerberos**

La provision a créé un fichier de configuration Kerberos dans /var/lib/samba/private/krb5.conf. Il faut le copier à l'emplacement standard attendu par les utilitaires système.

sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf

\# Vérification : le fichier doit ressembler à ceci :

cat /etc/krb5.conf

\# \[libdefaults\]

\# default_realm = MONLABO.LOCAL

\# dns_lookup_realm = false

\# dns_lookup_kdc = true

## **4.5 Configurer le DNS vers soi-même**

Maintenant que Samba est prêt à démarrer son DNS interne, on reconfigure resolv.conf pour que le serveur utilise son propre DNS AD, ce qui lui permettra de résoudre les enregistrements SRV du domaine.

sudo bash -c 'cat > /etc/resolv.conf << EOF

search monlabo.local

nameserver 192.168.56.10

EOF'

\# Protéger le fichier contre l'écrasement automatique :

sudo chattr +i /etc/resolv.conf

\# Pour modifier le fichier plus tard : sudo chattr -i /etc/resolv.conf

## **4.6 Démarrer et activer Samba AD DC**

Sur Ubuntu 24.04, le service Samba AD DC s'appelle samba-ad-dc - c'est différent de smbd qui est le Samba standalone. Ces deux services ne doivent pas tourner en même temps.

sudo systemctl unmask samba-ad-dc

sudo systemctl start samba-ad-dc

sudo systemctl enable samba-ad-dc

\# Vérification du statut :

sudo systemctl status samba-ad-dc

\# Doit afficher : active (running)

## **4.7 Tests de validation post-provision**

Ces tests sont indispensables - ne passe pas à la section suivante si l'un d'eux échoue. Chaque test vérifie un pilier différent de l'AD.

### **Test 1 - Niveau fonctionnel du domaine**

sudo samba-tool domain level show

\# Attendu : Domain Level: 2008 R2 (ou supérieur)

\# Forest Level: 2008 R2

### **Test 2 - DNS interne (enregistrements SRV)**

host -t SRV \_kerberos.\_udp.monlabo.local

\# Attendu : \_kerberos.\_udp.monlabo.local has SRV record ... dc1.monlabo.local.

host -t SRV \_ldap.\_tcp.monlabo.local

\# Attendu : \_ldap.\_tcp.monlabo.local has SRV record ... dc1.monlabo.local.

host -t A dc1.monlabo.local

\# Attendu : dc1.monlabo.local has address 192.168.56.10

### **Test 3 - Authentification Kerberos**

kinit <administrator@MONLABO.LOCAL>

\# Entre le mot de passe Administrator AD

klist

\# Attendu : ticket avec 'krbtgt/MONLABO.LOCAL@MONLABO.LOCAL'

kdestroy # Nettoyer le ticket de test

### **Test 4 - Accès LDAP**

samba-tool user list

\# Doit lister : Administrator, Guest, krbtgt

### **Test 5 - Accès aux partages**

smbclient -L localhost -U Administrator

\# Doit afficher : SYSVOL, NETLOGON

# **5\. Gestion des utilisateurs AD et des partages**

Maintenant que le domaine est opérationnel, on va créer des utilisateurs et des groupes dans l'Active Directory, puis configurer un partage Samba accessible uniquement aux membres d'un groupe AD.

## **5.1 Créer des utilisateurs dans l'AD**

Tous les utilisateurs AD sont gérés avec samba-tool sur le serveur. La syntaxe est similaire à celle des outils PowerShell Active Directory de Windows Server.

\# Créer un utilisateur 'alice' avec les attributs POSIX (UID/GID) :

sudo samba-tool user create alice 'MotDePasseAlice123!' \\

\--given-name='Alice' \\

\--surname='Dupont' \\

\--mail-address='<alice@monlabo.local>' \\

\--uid-number=10001 \\

\--gid-number=10001 \\

\--home-directory='/home/alice' \\

\--login-shell='/bin/bash'

\# Créer un deuxième utilisateur :

sudo samba-tool user create bob 'MotDePasseBob123!' \\

\--given-name='Bob' \\

\--surname='Martin' \\

\--uid-number=10002 \\

\--gid-number=10001 \\

\--home-directory='/home/bob' \\

\--login-shell='/bin/bash'

\# Lister les utilisateurs :

sudo samba-tool user list

**ℹ Note :** Les attributs --uid-number et --gid-number (RFC 2307) sont rendus disponibles grâce au flag --use-rfc2307 lors de la provision. Sans eux, les clients Linux ne pourraient pas mapper les utilisateurs AD à des UID Unix réels.

## **5.2 Créer des groupes AD**

\# Créer le groupe 'staff' :

sudo samba-tool group add staff \\

\--gid-number=10001 \\

\--nis-domain=monlabo

\# Ajouter alice et bob au groupe staff :

sudo samba-tool group addmembers staff alice

sudo samba-tool group addmembers staff bob

\# Vérifier :

sudo samba-tool group listmembers staff

## **5.3 Configurer un partage de fichiers Samba**

Le fichier smb.conf généré par la provision ne contient que SYSVOL et NETLOGON. On va y ajouter un partage 'shared' accessible au groupe staff.

\# Créer le dossier partagé avec les bonnes permissions :

sudo mkdir -p /srv/samba/shared

sudo chgrp staff /srv/samba/shared # Groupe propriétaire = staff AD

sudo chmod 2770 /srv/samba/shared # setgid : les nouveaux fichiers héritent du groupe

\# Activer les ACL POSIX sur le système de fichiers :

sudo tune2fs -l /dev/sda1 | grep 'Default mount'

\# Si 'acl' n'est pas listé, ajouter acl aux options de montage dans /etc/fstab

sudo nano /etc/samba/smb.conf

\# Ajouter à la FIN du fichier (ne pas modifier les sections existantes) :

\[shared\]

comment = Partage commun - Groupe staff

path = /srv/samba/shared

valid users = @staff

read only = no

browseable = yes

create mask = 0660

directory mask = 0770

force group = staff

sudo systemctl restart samba-ad-dc

\# Test d'accès depuis le serveur lui-même :

smbclient //dc1.monlabo.local/shared -U alice

\# Entrer le mot de passe d'alice → doit afficher l'invite smb: \\>

# **6\. Configuration du client (VM 2)**

On va maintenant configurer le client Ubuntu Desktop 24.04 pour qu'il rejoigne le domaine Active Directory. L'outil realmd rend cette opération remarquablement simple - il détecte automatiquement le domaine via DNS et configure SSSD, PAM et NSS en une seule commande.

## **6.1 Installation d'Ubuntu Desktop 24.04**

- Démarrer la VM sur l'ISO Ubuntu 24.04.4 Desktop.
- Choisir 'Installation normale' pour inclure GNOME et toutes les applications standard.
- Définir le hostname comme poste01.
- Créer un utilisateur local administrateur (ex : localadmin) - distinct des comptes AD.
- Terminer l'installation et démarrer sur le bureau GNOME.

## **6.2 Configurer l'IP fixe et le DNS**

Sur le client Desktop 24.04, NetworkManager gère le réseau par défaut. On peut configurer l'IP fixe directement depuis GNOME, ce qui est plus propre que d'écrire un fichier Netplan à la main (les deux coexisteraient et pourraient se contredire).

### **Via l'interface graphique GNOME (méthode recommandée)**

- Ouvrir Paramètres → Réseau.
- Cliquer sur l'engrenage à côté de l'interface Host-Only (enp0s8).
- Aller dans l'onglet IPv4.
- Choisir la méthode 'Manuel'.
- Adresse : 192.168.56.20 - Masque : 255.255.255.0 - Passerelle : (vide).
- Dans le champ DNS, entrer uniquement : 192.168.56.10 (l'adresse du contrôleur de domaine).
- Cliquer Appliquer, puis désactiver/réactiver l'interface.

**⚠ Important :** Le DNS du client DOIT pointer vers le contrôleur de domaine (192.168.56.10) et non vers 8.8.8.8. C'est le serveur AD qui résout les noms du domaine. Sans cela, realmd ne trouvera pas le domaine.

### **Vérification de la connectivité**

\# Depuis un terminal sur le client :

ping -c 3 192.168.56.10 # Ping le serveur AD

host monlabo.local 192.168.56.10 # Le DNS AD doit répondre

host dc1.monlabo.local 192.168.56.10 # Doit retourner 192.168.56.10

### **Mettre à jour /etc/hosts**

echo '192.168.56.10 dc1.monlabo.local dc1' | sudo tee -a /etc/hosts

## **6.3 Installer les paquets nécessaires**

sudo apt update

sudo apt install -y realmd sssd sssd-tools sssd-ad \\

adcli krb5-user samba-common-bin packagekit \\

libnss-sss libpam-sss oddjob oddjob-mkhomedir

\# Lors de l'installation de krb5-user :

\# Realm par défaut → MONLABO.LOCAL (MAJUSCULES)

\# Serveur KDC → dc1.monlabo.local

\# Serveur admin → dc1.monlabo.local

## **6.4 Découverte et jonction du domaine avec realmd**

realmd est un service qui automatise la jonction à un domaine AD. Il interroge le DNS pour trouver les enregistrements SRV du domaine, négocie avec le KDC Kerberos, et configure SSSD automatiquement.

### **Étape 1 - Découverte du domaine**

sudo realm discover monlabo.local

\# Sortie attendue :

\# monlabo.local

\# type: kerberos

\# realm-name: MONLABO.LOCAL

\# domain-name: monlabo.local

\# configured: no

\# server-software: active-directory

\# client-software: sssd

Si la découverte échoue, c'est un problème DNS - vérifie que le client utilise bien 192.168.56.10 comme DNS et que samba-ad-dc tourne sur le serveur.

### **Étape 2 - Joindre le domaine**

sudo realm join --user=Administrator monlabo.local

\# Entrer le mot de passe Administrator AD quand demandé

\# Vérification de la jonction :

sudo realm list

\# Doit afficher : configured: kerberos-member

**✔ Conseil :** La commande realm join fait tout automatiquement : elle crée un compte machine dans l'AD (poste01\$), génère un fichier /etc/sssd/sssd.conf complet, configure /etc/nsswitch.conf, et active PAM. C'est l'un des grands avantages de realmd sur la configuration manuelle.

## **6.5 Configurer la création automatique de répertoires home**

Par défaut, quand un utilisateur AD se connecte pour la première fois, son répertoire /home n'existe pas encore. Le service oddjobd crée automatiquement ce répertoire à la première connexion.

sudo pam-auth-update --enable mkhomedir

sudo systemctl enable --now oddjobd

\# Alternative si oddjob pose problème : configurer PAM manuellement

\# sudo nano /etc/pam.d/common-session

\# Ajouter à la fin : session optional pam_mkhomedir.so skel=/etc/skel umask=077

## **6.6 Affiner la configuration SSSD**

realmd génère un sssd.conf fonctionnel, mais quelques ajustements le rendent plus agréable à utiliser au quotidien, notamment pour la syntaxe de connexion (taper 'alice' plutôt que '<alice@monlabo.local>').

sudo nano /etc/sssd/sssd.conf

Les paramètres importants à vérifier ou ajouter dans la section \[domain/monlabo.local\] :

\[sssd\]

domains = monlabo.local

config_file_version = 2

services = nss, pam

\[domain/monlabo.local\]

default_shell = /bin/bash

krb5_store_password_if_offline = true

cache_credentials = true

krb5_realm = MONLABO.LOCAL

realmd_tags = manages-system joined-with-adcli

id_provider = ad

fallback_homedir = /home/%u

ad_domain = monlabo.local

use_fully_qualified_names = false # Permet de taper 'alice' au lieu de '<alice@monlabo.local>'

ldap_id_mapping = false # Utilise les UID/GID RFC 2307 qu'on a définis

access_provider = ad

sudo chmod 600 /etc/sssd/sssd.conf # SSSD refuse de démarrer si droits incorrects

sudo systemctl restart sssd

## **6.7 Tests de validation sur le client**

### **Test 1 - Résolution des utilisateurs AD**

getent passwd alice

\# Attendu : alice:\*:10001:10001:Alice Dupont:/home/alice:/bin/bash

id alice

\# Attendu : uid=10001(alice) gid=10001(staff) groupes=10001(staff),...

### **Test 2 - Authentification Kerberos depuis le client**

kinit <alice@MONLABO.LOCAL>

klist

\# Doit afficher un ticket Kerberos valide pour alice

### **Test 3 - Connexion graphique avec un compte AD**

Sur l'écran de connexion GDM (l'écran de verrouillage GNOME), il y a un champ 'Autre utilisateur' en bas. Entrer 'alice' comme identifiant et son mot de passe AD. GNOME doit créer /home/alice et ouvrir la session.

**ℹ Note :** Si la connexion graphique échoue mais que 'id alice' fonctionne dans le terminal, le problème vient souvent de PAM. Vérifier avec : sudo pam-auth-update --force

# **7\. Accès aux partages Samba depuis le client**

Il existe deux façons d'accéder aux partages depuis le client : via l'interface graphique Nautilus (pour les utilisateurs), et via la ligne de commande avec montage permanent dans /etc/fstab (pour les administrateurs ou les besoins d'automatisation).

## **7.1 Accès graphique via Nautilus**

GNOME Files (Nautilus) supporte nativement le protocole SMB. L'accès est particulièrement fluide quand l'utilisateur est connecté avec son compte AD car SSSD peut passer les credentials Kerberos automatiquement.

- Ouvrir l'application Fichiers (Nautilus) depuis le dock GNOME.
- Dans la barre latérale gauche, cliquer sur Autres emplacements tout en bas.
- Dans la barre en bas 'Se connecter au serveur', taper : smb://dc1.monlabo.local/shared
- Cliquer Connexion. Dans la boîte de dialogue d'authentification :

- Choisir : Utilisateur enregistré
- Domaine : MONLABO
- Nom d'utilisateur : alice
- Mot de passe : le mot de passe AD d'alice

- Le partage s'ouvre comme un dossier local. Pour le mettre en favori : Ctrl+D.

**✔ Conseil :** Si Nautilus ne propose pas l'option SMB, installe le backend gvfs : sudo apt install -y gvfs-backends gvfs-fuse, puis déconnecte-toi et reconnecte-toi à la session GNOME.

## **7.2 Montage permanent via /etc/fstab avec authentification Kerberos**

Pour un montage automatique au démarrage qui utilise un ticket Kerberos plutôt qu'un mot de passe en clair dans un fichier, on utilise l'option sec=krb5 de cifs-utils. C'est la méthode la plus sécurisée.

sudo apt install -y cifs-utils

\# Créer le point de montage :

sudo mkdir -p /mnt/shared

\# Ajouter dans /etc/fstab :

sudo nano /etc/fstab

\# Montage Samba avec authentification Kerberos (ajouter à la fin) :

//dc1.monlabo.local/shared /mnt/shared cifs sec=krb5,uid=10001,gid=10001,\_netdev,nofail 0 0

\# Alternative avec fichier credentials si Kerberos pose problème :

//dc1.monlabo.local/shared /mnt/shared cifs credentials=/etc/samba/creds-alice,\_netdev,nofail 0 0

\# Si tu utilises l'option credentials, créer le fichier :

sudo bash -c 'cat > /etc/samba/creds-alice << EOF

username=alice

password=MotDePasseAlice123!

domain=MONLABO

EOF'

sudo chmod 600 /etc/samba/creds-alice

\# Tester le montage sans redémarrer :

sudo mount -a

ls /mnt/shared

**⚠ Important :** L'option \_netdev est indispensable - elle indique au système d'attendre que le réseau soit disponible avant de monter ce partage. Sans elle, la VM peut se bloquer au démarrage si le serveur n'est pas encore joignable.

# **8\. Administration quotidienne de l'Active Directory**

Une fois le domaine en place, voici les opérations courantes que tu effectueras régulièrement via samba-tool sur le serveur.

## **8.1 Gestion des utilisateurs**

\# Lister tous les utilisateurs :

sudo samba-tool user list

\# Voir les détails d'un utilisateur :

sudo samba-tool user show alice

\# Réinitialiser un mot de passe :

sudo samba-tool user setpassword alice

\# Désactiver un compte (sans le supprimer) :

sudo samba-tool user disable alice

\# Réactiver un compte :

sudo samba-tool user enable alice

\# Supprimer un utilisateur :

sudo samba-tool user delete alice

## **8.2 Gestion des groupes**

\# Lister tous les groupes :

sudo samba-tool group list

\# Lister les membres d'un groupe :

sudo samba-tool group listmembers staff

\# Ajouter un membre à un groupe :

sudo samba-tool group addmembers staff bob

\# Retirer un membre d'un groupe :

sudo samba-tool group removemembers staff bob

## **8.3 Gestion des machines membres du domaine**

\# Lister les machines jointes au domaine :

sudo samba-tool computer list

\# Supprimer une machine du domaine (depuis le serveur) :

sudo samba-tool computer delete POSTE01\$

\# Pour retirer proprement une machine depuis le client :

sudo realm leave monlabo.local

## **8.4 Sauvegarde de l'Active Directory**

La sauvegarde d'un AD Samba se fait avec samba-tool domain backup. C'est la méthode recommandée car elle gère les aspects transactionnels de la base LDB.

\# Créer un backup complet de l'AD :

sudo samba-tool domain backup online \\

\--targetdir=/var/backups/samba \\

\--server=dc1.monlabo.local \\

\-U Administrator

\# Planifier une sauvegarde quotidienne avec cron :

echo '0 3 \* \* \* root samba-tool domain backup online --targetdir=/var/backups/samba --server=dc1.monlabo.local -U Administrator --password=VotreMotDePasseAdmin123!' | sudo tee /etc/cron.d/samba-backup

# **9\. Dépannage**

Les problèmes avec Active Directory ont souvent une chaîne causale : DNS → Kerberos → LDAP → SSSD → Samba. Travailler toujours dans cet ordre lors du diagnostic - un problème DNS rendra tous les autres inutiles à déboguer.

| **Symptôme**                                      | **Cause probable**                              | **Solution**                                                                             |
| ------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------- |
| realm discover ne trouve pas le domaine           | DNS client mal configuré                        | Vérifier que 192.168.56.10 est le DNS du client et que samba-ad-dc tourne sur le serveur |
| realm join échoue avec 'Insufficient permissions' | Mauvais mot de passe Administrator              | Tester : kinit <administrator@MONLABO.LOCAL> sur le client                               |
| getent passwd alice retourne rien                 | use_fully_qualified_names = true dans sssd.conf | Tester : getent passwd <alice@monlabo.local> ou changer à false                          |
| Connexion GNOME impossible avec compte AD         | PAM ou oddjob non configuré                     | Lancer : sudo pam-auth-update --force, vérifier oddjobd                                  |
| samba-ad-dc ne démarre pas                        | systemd-resolved encore actif sur port 53       | sudo systemctl disable --now systemd-resolved et relancer                                |
| Les tickets Kerberos expirent trop vite           | Désynchro horaire entre serveur et client       | Installer ntp sur les deux VMs : sudo apt install -y ntp                                 |
| Nautilus ne voit pas les partages SMB             | gvfs-backends manquant                          | sudo apt install -y gvfs-backends puis reconnexion GNOME                                 |
| fstab bloque le démarrage                         | Option \_netdev manquante                       | Ajouter \_netdev et nofail aux options de montage fstab                                  |
| SSSD cache des données obsolètes                  | Cache SSSD pas rafraîchi                        | sudo sss_cache -E puis sudo systemctl restart sssd                                       |

## **9.1 Commandes de diagnostic essentielles**

### **Sur le serveur**

sudo systemctl status samba-ad-dc

sudo samba-tool domain level show

sudo samba-tool drs showrepl # Vérifier la réplication

sudo journalctl -u samba-ad-dc -f # Logs en temps réel

host -t SRV \_kerberos.\_udp.monlabo.local # Test DNS SRV

testparm # Valider smb.conf

### **Sur le client**

sudo realm list # Statut de la jonction

sudo systemctl status sssd

sudo sss_cache -E # Vider le cache SSSD

sudo journalctl -u sssd -f # Logs SSSD en temps réel

id alice # Tester la résolution d'un utilisateur AD

kinit <alice@MONLABO.LOCAL> && klist # Tester l'auth Kerberos

# **10\. Aide-mémoire - Commandes essentielles**

| **Action**                      | **Commande (sur le serveur sauf mention)**                          |
| ------------------------------- | ------------------------------------------------------------------- |
| Statut du DC                    | sudo systemctl status samba-ad-dc                                   |
| Redémarrer Samba AD             | sudo systemctl restart samba-ad-dc                                  |
| Lister les utilisateurs AD      | sudo samba-tool user list                                           |
| Créer un utilisateur            | sudo samba-tool user create alice 'Pass123!' --uid-number=10003 ... |
| Réinitialiser un mot de passe   | sudo samba-tool user setpassword alice                              |
| Désactiver un compte            | sudo samba-tool user disable alice                                  |
| Lister les groupes              | sudo samba-tool group list                                          |
| Ajouter au groupe               | sudo samba-tool group addmembers staff alice                        |
| Lister les machines du domaine  | sudo samba-tool computer list                                       |
| Test DNS SRV Kerberos           | host -t SRV \_kerberos.\_udp.monlabo.local                          |
| Test ticket Kerberos (serveur)  | kinit <administrator@MONLABO.LOCAL> && klist                        |
| Tester résolution user (client) | getent passwd alice (sur le client)                                 |
| Vider cache SSSD (client)       | sudo sss_cache -E (sur le client)                                   |
| Joindre le domaine (client)     | sudo realm join --user=Administrator monlabo.local                  |
| Quitter le domaine (client)     | sudo realm leave monlabo.local                                      |
| Sauvegarde complète AD          | sudo samba-tool domain backup online --targetdir=/var/backups ...   |

_- Fin du guide -_

Ubuntu 24.04.4 LTS • Samba 4 AD DC • Kerberos • realmd • SSSD
