#  Examen Administration système 16 janvier 2026

## Les deux familles

les principales distributions de la famille Debian sont : 
- Debian
-  Ubuntu et ses variantes (Kubuntu, Xubuntu, Lubuntu)
-  Linux Mint
-  Kali linux

Les distributions Debian sont plus connue pour leur stabilité, il s'agit d'un logiciel libre pur qui possède une grande communauté et offre un support communautaire gratuit conçu pour les utilisateurs de bureau et les particuliers.
De plus Debian utilise un système de paquets  dkpg/apt.

et les principales distributions de la familles RedHat :
- Fedora
-  Red hat entreprise linux (RHEL)
-  Rocky linux
-  Alma linux
-  Cent OS
-  Oracle Linux

RedHat a un positionnement orienter entreprise et les environnement de serveur. Il s'agit d'une distribution d'entreprise payante accompagné d'un support technique complet qui garanti plus une stabilité pour les entreprises et possède des certifications professionnels.
Red Hat utilise npm pour leur système de gestion de paquets 

les autres distributions linux :
- Slackware : elle est connu pour suivre au mieux la "philosophie Unix"
-  SUSE linux : SUSE Linux Enterprise, connu pour son outil de configuration YaST (outil d'admin graphique)
-  Arch linux : distribution de type "rolling release", minimaliste et pour utilisateurs avancés
-  Gentoo : compilation depuis les sources qui permet une optimisation optimal 
- Alpine Linux : connu pour sa légèreté, son système de conteneurs et sa sécurité



## Back to basics

Différences fontamentalz en mode Kernel et mode utilisateur :
-**mode Kernel**: Le code s'exécute avec tous les privilèges avec un accèes direct avec le matériel ainsi qu'à toute la mémoire et aux instructions privilégié de l'ordinateur. Donc lorsqu'un code est éxécuté , une erreur peut faire crash tout le système.
-**mode utilisateur**  : le code s'éxécute dans un espace mémoire isolé et donc protégé, avec sans aucune accès au matériel et toute commande sensible nécessité donc un appel système qui le bascule donc temporairement dans le mode kernel. Vu que le code est exécuté de manière isolé une erreur ne pourra faire crash que le processus concerné 


## Qui est où ?

- **/etc:** contient tous les fichiers de configurations système et des application
- **/bin et /user/bin:** il contient les binaire essentiels et accessibles pour tout les utilisateurs. /bin contient les commandes de base nécessaires au démarrage et /user/bin contient les programmes des utilisateurs 
- **/sbin et /usr/sbin:** Binaires système réservés à l'administration (root). */sbin* contient les outils essentiels (fsck, init, reboot).  */usr/sbin* contient les démons et outils d'admin (apache2, sshd, useradd). Fusion fréquente également. 
- **/home:** Répertoires personnels des utilisateurs.
- **/var:** données variables, les fichiers changent durant l'exécution du système (logs, caches, bases de données, etc.).
- **/var/log:** logs des système et des applications 
- **/var/lib:** contient les données de type persistantes des applications comme par exemple les bases de données 

## Où est le pilote ?

**Liste des pilotes (modules) chargés en mémoire :**
```bash
lsmod
```

**La liste des fichiers binaires  (modules) qui les fournissent ?**
```bash
ls /lib/modules/$(uname -r)/kernel/drivers/
```
ou 
```bash 
modinfo nom_module
```


**Forcer le chargement d'un module :**
```
modprobe nom_module
```

**Déchargement d'un module :**
```bash
modprobe -r nom_module
```
**Les messages qu'ils ont émis ?**
```bash 
dmesg
```
ou 
```bash
journalctl -k
```

## MS Windows

**Fonctionnement de WSL (Windows Subsystem for Linux) :**
Il s'agit d''un véritable kernel linux complet et certifié tournant dans une machine virtuelle, son architecture est donc basé sur la virtualisation Les commandes Linux s'exécutent directement grâce à un noyau dédié qui permet de garantir la sécurité

Donc il permet :
- D'éxécuter des distributions Linux complètes
- Utiliser des outils en ligne de commande Linux
- Le développement multi- plateforme 
- Accès au système de fichiers Windows depuis Linux et vice-versa

**Protocoles utilisé :**
-   **9P protocol** : Partage de fichiers entre Windows et Linux (WSL 2)
-   **Hyper-V** : Virtualisation légère (WSL 2)
-   **Réseau virtuel** : NAT pour la mise en réseau
-   **Plan 9 filesystem protocol** : Accès /mnt/c/ aux disques Windows

## On commence

Commencer par installer ssh 
```bash
sudo apt install openssh-server
```

modifier le fichier "_/etc/ssh/sshd_config_"
```bash
sudo vi /etc/ssh/sshd_config
```
 le port d'écoute SSH est 22, changez-le afin de brouiller les pistes. Pour cela, modifiez le paramètre "Port" du fichier de configuration :
 ```bash
port 2021
 ```
 
 Pour que SSH écoute seulement sur l'adresse IP "_192.168.175.129_"
 ```bash
 ListenAddress 192.168.175.129
 ```
 
redémarrez le service
```bash
sudo systemctl restart sshd
```

puis on peut configurer Configurer le timeout des sessions SSH:
```bash
ClientAliveInterval 600
ClietAliveCountMax 0
```

puis on peut autoriser seulement la connexion pour un ou plusieurs utilisateurs. Toujours dans /etc/ssh/sshd_config

```bash
AllowUsers [nom utilisateur]
```
ou par groupe 
```bash
AllowGroups [MonGroupe]
```

redémarrer ssh
```bash
systemctl restart ssh
```

Maintenant on va configurer une authentification par clé :
créez la paire de clés en utilisant la commande suivante

```bash
ssh-keygen -b 8192
```

copier votre clé publique vers le serveur SSH

```bash
ssh-copy-id -p 2021 tintin@192.168.175.129
```

vérifier que votre clé est bien arrivée sur le serveur SSH
```bash
ssh -p 2021 tintin@192.168.175.129
```

et enfin des&ctiver l'authentification par mot de passe classique
```bash
sudo vi /etc/ssh/sshd_config
```
```bash
PasswordAuthentication no
```

redémarrer ssh
```bash
systemctl restart ssh
```



## Alerte !

Tout d'abord ne pas ignorer le message et vérifier la possible cause :
- le serveur a peut être été reinstallé 
- les clé ont peut être été régénérées
- mis a jour ? 

Si vous n'êtes pas admin contacter l'admin pour confirmmer un changement légitime 

vérifier l'empreinte de la nouvelle clé 
```bash
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub 
ssh-keygen -lf /etc/ssh/ssh_host_rsa_key.pub
```

Si le changement est légitime et confirmé 
```bash
ssh-keygen -R hostname
   vi ~/.ssh/known_hosts
```

 

## Oh les gourmands !

**pour identifier les utilisateurs consommant le plus d'espace de stockage :**
```bash
sudo  du -sh /home/* |  sort -hr |  head -n 10
```

## On va aider les devs...

**Sur Debian:** 
```bash
sudo  apt update 
sudo  apt  install build-essential

#verification
gcc --version 
make --version
```


**Sur Red Hat/Fedora/CentOS :**
```bash
sudo dnf groupinstall "Development Tools"

#verification
gcc --version 
make --version
```
## Qui fait quoi ?

Retrouver les dernières connexions utilisateurs :
```bash
last
last -10 #10 dernières connexions
```

Identifier les sessions qui a utilisé de sudo :
```bash
grep  "sudo:" /var/log/auth.log
```

## Post mortem

Nous pouvons tout d'abord essayer de voir les log des boot précedents pour verifier donc le journal systeme
```bash
sudo journalctl -b -1 # boot précédent
sudo journalctl --list-boots # tousles boots
```

Le Kernel a surement du renvoyer un message d'erreur, on part le vérifier 
```bash
sudo journalctl -k -b -1
```

on peut rechercher les erreurs critiques
```bash
sudo journalctl -p err -b -1
```




## On surveille...

Sur un serveur GNU/Linux qui démarre via _systemd_, un service nommé _bidule_

Savoir si le service est lancé au démarrage automatiquement:
```bash
sudo systemctl is-enabled bidule
```

 Savoir s'il est en cours de fonctionnement
 ```bash
 sudo systemctl status bidule
 ```

Faire en sorte qu'il soit lancé:
```bash
sudo systemctl enable bidule # Activer au démarrage
sudo systemctl start bidule # Démarrer immédiatement
```

Examiner les messages qu'il émet:
```bash
sudo journalctl -u bidule -f
```

Redémarrer le service
```bash
sudo systemctl restart bidule
```

Déterminer de quel paquet il provient:
- Debian
```bash
dpkg -S $(which bidule)
```

- Red Hat
```bash
rpm -qf $(which bidule)
```


## Zut

1. Redémarrer la machine et accéder à GRUB (en appuyant sur esp au démarrage)
2. appuyez sur "e" pour editer quand vous êtes sur la ligne démarrage kernel 
3. trouver la ligne "linux" et modifier la fin de la ligne:
```bash
rw init=/bin/bash
```
4. le booter en appuyant sur ctrl+x
5. a ce moment la on se retrouve dans le shell du root et depuis ce shell on peut par exemple changer le mot de passe du root 
```bash
passwd root
```
6. remonter le tout 
```bash
mount -o remount,rw /
exec /sbin/init
```



## C'est la faute à Rémy

1. On peut vérifier l'etat du service web 
```bash
sudo systemctl status apache2
sudo systemctl status nginx
```
On utilise systemctl pour vérifié si le service tourne, à affiche les erreurs récentes et l'état d'activation.

2. On peut aussi vérifier les ports d'écoute 
```bash
sudo ss -tulpn |  grep :80 #http
sudo ss -tulpn |  grep :443 #https
```
ss vérifie que le service écoute sur les bons ports. Si aucun processus n'écoute sur les ports 80/443 c'est que le service n'a pas démarré correctement.

3. on peut aussi essayer de tester la connectivité réseau
on test localement
```bash
curl -I http://localhost 
curl -I https://localhost
```
curl teste la connectivité HTTP.  -I récupère juste les headers. on veut juste vérifie si le problème est local ou si il vient du réseau.

4. Le pare feu peut aussi être le problème 
```bash
sudo iptables -L -n -v |  grep -E '80|443'
```
Les règles iptables qui gere les règles des pares feu peuvent empêcher les connexions entrantes.

5. Vérifier les logs d'erreur peu être utile
```bash
sudo  tail -f /var/log/apache2/error.log
```
Avec tail on vérifie dans le fichier error.log qui regroupe toute les erreurs. Le diagnostic peut ainsi simplifier si on a le log de l'erreur

6. On peut vérifier les certificat ssl
```bash
openssl s_client -connect localhost:443
```
openssl vérifie validité certificats, dates expiration, chaîne de certification si le certificat est expiré alors forcement il y aura une erreur HTTPS.


## Au charbon !

Pour installer un serveur de base de donnée Elastic sur un système Debian on va se référer a la documentation officiel de Elastic 

[Install Elasticsearch with a Debian package | Elastic Docs](https://www.elastic.co/docs/deploy-manage/deploy/self-managed/install-elasticsearch-with-debian-package)

Avant de commencer mettons a jour nos paquets
```bash
sudo  apt update
```

Importons le "Elasticsearch PGP key"
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

installons maintenant le paquet Debian Elasticsearch, dans notre cas on va passer par le APT repository
- 1. on doit d'abord installer le paquet apt-transport-https
```bash
sudo apt-get install apt-transport-https
```
- 2. Ajouter le dépôt APT
```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
```
- 3. Et enfin installer elasticsearch
```bash
sudo  apt update
sudo apt-get update && sudo apt-get install elasticsearch
```
**Passons à la configuration** 

- 1. ouvrons le fichier de configuration 
```bash
vi elasticsearch.yml
```
- 2. Decommenter la ligne #cluster.name: my-application et lui donner le nom que l'on souhaite
```bash
cluster.name: elasticsearch-demo
```
- 3. Decommenter #network.host: 192.168.0.1 et mettre 0.0.0.0

- 4. Decommenter #transport.host: 0.0.0.0 pour permettre a Elasticsearch d'ecouter sur tous les interfaces réseau 

## Le Web

