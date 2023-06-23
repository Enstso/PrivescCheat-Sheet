# Privesc-cheat-sheet

Important si l'on éxécute un script pour être sur de garder les droits de root il faut rajouter le flag -p.

## Enumération

- Utiliser LinEnum sur la machine cible.
- Sudo -l pour répertorier les commandes que vous pouvez utiliser en tant que super utilisateur.

### La sortie de LinEum

Noyau Les informations sur le noyau sont affichées ici. Il y a très probablement un exploit du noyau disponible pour cette machine.

Pouvons-nous lire/écrire des fichiers sensibles : les fichiers accessibles en écriture par tous sont indiqués ci-dessous. Ce sont les fichiers que tout utilisateur authentifié peut lire et écrire. En examinant les autorisations de ces fichiers sensibles, nous pouvons voir où il y a une mauvaise configuration qui permet aux utilisateurs qui ne devraient généralement pas être en mesure de pouvoir écrire sur des fichiers sensibles.

Fichiers SUID : la sortie des fichiers SUID est affichée ici. Il y a quelques éléments intéressants que nous allons certainement examiner comme un moyen d'augmenter les privilèges. SUID (Set owner User ID up on execution) est un type spécial d'autorisations de fichier accordées à un fichier. Il permet au fichier de s'exécuter avec les autorisations de son propriétaire. S'il s'agit de root, il s'exécute avec les autorisations root. Cela peut nous permettre d'augmenter les privilèges. 

Contenu de Crontab : les tâches cron planifiées sont présentées ci-dessous. Cron est utilisé pour programmer des commandes à un moment précis. Ces commandes ou tâches planifiées sont appelées "tâches cron". Liée à cela, la commande crontab crée un fichier crontab contenant des commandes et des instructions à exécuter par le démon cron. Il y a certainement suffisamment d'informations pour justifier une tentative d'exploitation de Cronjobs ici.

etc...

### énumération manuelle

find / -perm -u=s -type f 2>/dev/null

find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null


## Exploitation

En fonction des vulnérabilités trouvé, nous pouvons aussi rechercher des exploits sur exploit-db.

Utiliser ce site pour trouver des exploits :
https://gtfobins.github.io

### Exploiter le fichier /etc/passwd 

exemple :

    test:x:0:0:root:/root:/bin/bash

nom d'utilisateur:mdp:idUser:idGroup:infoUser:répertoireUser:commande

Si nous avons des droits d'écriture nous pouvons modifier des utilisateurs ou encore ajouter.

Pour générer un mot de passe valide :

openssl passwd motDepasse

### Exploiter vim/vi 

Si vim a mal été configuré et qu'il est possible de lancer vim en tant que root, nous pouvons tenter d'échapper à vim.

- Lancer VIM
- Tenter d'avoir un shell avec la commande !:sh
- Chercher un exploit sur https://gtfobins.github.io/

### Exploiter cron

- Voir le contenu du fichier /etc/crontab

Le format cronjob :

#id|m=minutes|h =heure|dom=jour du mois|mon=mois|dow = jour de la semaine|user=qui l'executera|commande

Si nous avons des droits d'écriture et que le propriétaire est root ou une autre personne, nous pouvons modifer le contenu du fichier pour récupérer un shell.


### Exploiter la variable $PATH 

PATH est une variable d'environnement dans les systèmes d'exploitation de type Linux et Unix qui spécifie les répertoires contenant les programmes exécutables. Lorsque l'utilisateur exécute une commande dans le terminal, il recherche des fichiers exécutables à l'aide de la variable PATH en réponse aux commandes exécutées par un utilisateur.

 nous avons un binaire SUID. En l'exécutant, nous pouvons voir qu'il appelle le shell système pour effectuer un processus de base comme les processus de liste avec "ps". Contrairement à notre précédent exemple SUID, dans cette situation, nous ne pouvons pas l'exploiter en fournissant un argument pour l'injection de commande, alors que pouvons-nous faire pour essayer d'exploiter cela ?

Nous pouvons réécrire la variable PATH à l'emplacement de notre choix ! Ainsi, lorsque le binaire SUID appelle le shell système pour exécuter un exécutable, il en exécute un que nous avons écrit à la place !

Comme pour tout fichier SUID, il exécutera cette commande avec les mêmes privilèges que le propriétaire du fichier SUID ! S'il s'agit de root, en utilisant cette méthode, nous pouvons exécuter toutes les commandes que nous aimons en tant que root 

### Exploitation /etc/shadow


Si nous pouvons voir son contenu, nous pouvons récupérer les hachs et les cracker via john the ripper.

Rappel l'algorithme du fichier est le sha512crypt

Nous pouvons aussi le modifier en changeant le mdp, pour cela il faudra le hasher.

mkpasswd -m sha-512 motDePasse

### Exploitation des fichiers d'historique

Affichez le contenu de tous les fichiers d'historique cachés dans le répertoire personnel de l'utilisateur :

cat ~/.*history | less

### Exploitation des fichiers de configuration

par exemple un fichier : .ovpn

### Exploitation des clés SSH :

Si un utilisateur n'a pas sécurisé le répertoire dans lequel se trouve le dossier /.ssh est qu'on a les permissions pour voir le contenu de tous les fichiers.

Si nous récupérons une clé privée ssh, nous pourrons nous connecter.

Avant ça il est important de changer les droits de la clé sinon le client ssh refusera de les utiliser :

chmod 6000 key_ssh

puis connetons nous avec la clé :

ssh -i key_ssh user@ip

OU 

ssh -i key_ssh -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa user@ip 

Il existe d'autres outils pour escalation des privileges :

Linpeas et lse.sh