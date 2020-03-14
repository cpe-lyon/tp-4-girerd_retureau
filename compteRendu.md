# Compte-Rendu TP4 - Utilisateurs, groupes et permissions
 
&nbsp;

 ## Auteurs et date
 *Thomas GIRERD*
 
 *Guillaume RETUREAU*
 
*Date : 13.03.2020*

&nbsp;

***

&nbsp;

### Exercice 1. Gestion des utilisateurs et des groupes

**1. Commencez par créer deux groupes groupe1 et groupe2**

```
# addgroup groupe1

Adding group `groupe1' (GID 1001) ...
Done.

# addgroup groupe2

Adding group `groupe2' (GID 1002) ...
Done.
```

&nbsp;

**2. Créez ensuite 4 utilisateurs u1, u2, u3, u4 avec la commande useradd, en demandant la création de leur dossier personnel et avec bash pour shell**

```
# useradd -m u1
# useradd -m u2
# useradd -m u3
# useradd -m u4
```

*Note: l'option -m permet de créer le dossier utilisateur*

&nbsp;

**3. Ajoutez les utilisateurs dans les groupes créés :**
- **u1, u2, u4 dans groupe1**
- **u2, u3, u4 dans groupe2**

```
# usermod -a -G groupe1 u1
# usermod -a -G groupe1 u2
# usermod -a -G groupe1 u4

# usermod -a -G groupe2 u2
# usermod -a -G groupe2 u3
# usermod -a -G groupe2 u4
```
`usermod -a -G nom_groupe nom_utilisateur` 
*Permet d'ajouter un utilisateur existant à un groupe (secondaire) existant, càd en conservant ses groupes*
&nbsp;

**4. Donnez deux moyens d’afficher les membres de groupe2**

```
# grep groupe1 /etc/group | cut -d: -f4
u1,u2,u4
```
On installe le package **members** (`apt install members`)
```
# members groupe1
u1 u2 u4
```

&nbsp;

**5. Faites de groupe1 le groupe propriétaire de /home/u1 et /home/u2 et de groupe2 le groupe propriétaire de /home/u3 et /home/u4**

```
# chgrp groupe1 /home/u1
# chgrp groupe1 /home/u2
# chgrp groupe2 /home/u3
# chgrp groupe2 /home/u4
```
*Note: On pourrait aussi utiliser l'option -R pour affecter les fichiers et sous dossiers (récursif)*

&nbsp;

**6. Remplacez le groupe primaire des utilisateurs :**
- **groupe1 pour u1 et u2**
- **groupe2 pour u3 et u4**

```
# usermod u1 -g groupe1
# usermod u2 -g groupe1
# usermod u3 -g groupe2
# usermod u4 -g groupe2
```
A ce stade, on vérifie les groupes de nos utilisateurs:
```
# groups u1
u1 : groupe1
# groups u2
u2 : groupe1 groupe2
# groups u3
u3 : groupe2
# groups u4
u4 : groupe2 groupe1
```

&nbsp;

**7. Créez deux répertoires /home/groupe1 et /home/groupe2 pour le contenu commun aux groupes, et mettez en place les permissions permettant aux membres de chaque groupe d’écrire dans le dossier associé.**

```
# mkdir /home/groupe1
# chgrp groupe1 /home/groupe1
# chmod g+w /home/groupe1
# mkdir /home/groupe2
# chgrp groupe2 /home/groupe2
# chmod g+w /home/groupe2
```
*On crée le dossier, transfert l'ownership du groupe, puis donne les droit d'écriture au groupe*
&nbsp;


**8. Comment faire pour que, dans ces dossiers, seul le propriétaire d’un fichier ait le droit de renommer ou supprimer ce fichier ?**

https://stackoverflow.com/questions/1163294/changing-chmod-for-files-but-not-directories

`find . -type f -print0 | xargs -0 chmod 644`

*Permet de changer les permissions des fichiers uniquement, en conservant les permissions des dossieres appliquées précédemment*
&nbsp;


**9. Pouvez-vous vous connecter en tant que u1 ? Pourquoi ?**

On ne peut pas se connecter en tant que u1 car son compte n'est pas actif (il n'a pas de mot de passe).
Néamoins on peut simuler le fait d'être un utilisateur en utilisant la commande:

`su - u1`

&nbsp;

**10. Activez le compte de l’utilisateur u1 et vérifiez que vous pouvez désormais vous connecter avec son compte**

```
# passwd u1
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
```

&nbsp;


**11. Quels sont l’uid et le gid de u1 ?**

On peut afficher l'uid et gid d'un utilisateur:
```
# id u1
uid=1001(u1) gid=1001(groupe1) groups=1001(groupe1)
```

Son uid est **1001**
Son gid est **1001**
&nbsp;


**12. Quel utilisateur a pour uid 1003 ?**

```
# cat /etc/passwd | cut -d: -f1,3 | grep 1003 | cut -d: -f1
u3
```
`cat /etc/passwd` Permet de récupérer le contenu du fichier /etc/passwd
`cut -d: -f1,3` Permet de récupérer les colonnes 1 (utilisateur) et 3 (uid)
`grep 1003` Permet de récupérer les lignes contenant 1003
`cut -d: -f1` Permet de récupérer la 1ère colonne -> le nom d'utilisateur


&nbsp;


**13. Quel est l’id du groupe groupe1 ?**

De même que précédemment (sur le fichier /etc/group), avec des colonnes différentes, car il n'y a pas de commande comme `id` pour les groupes.
```
# cat /etc/group | cut -d: -f1,3 | grep groupe1 | cut -d: -f2
1001
```
Le groupe **groupe1** a pour gid **1001**
&nbsp;


**14. Quel groupe a pour guid 1002 ? ( Rien n’empêche d’avoir un groupe dont le nom serait 1002...)**

```
# cat /etc/group | cut -d: -f3 | grep -n 1002 | cut -d: -f1
groupe2
```
*Note: Cette solution de prends pas en compte le cas où un groupe possède le gid recherché en tant que nom*

On pourrait le traiter comme celà:

```
cat /etc/group | head -`cat /etc/group | cut -d: -f3 | grep -n 1002 | cut -d: -f1` | tail -1 | cut -d: -f1
groupe2
```
On réutilise la commande précédente, rejoutant -n à grep:

`cat /etc/group | cut -d: -f3 | grep -n 1002 | cut -d: -f1` Permet d'obtenir le numéro de la ligne du groupe ayant le gid 1002

On l'utilise ensuite en paramètre de head (`head -{numéro de ligne}`)

Finalement:
`cat /etc/group | head -n | tail -1 | cut -d: -f1` Permet de retourner la 1ère colonne de la nième ligne (n obtenu via la commande précédemment décrite), ce qui correspond au nom de l'utilisateur de gid **1002**

*Note: On pourrait utiliser la même methode pour éviter les erreur de recherche avec les commandes précédentes*


&nbsp;


**15. Retirez l’utilisateur u3 du groupe groupe2. Que se passe-t-il ? Expliquez**

```
# groups u3
u3 : groupe2

# gpasswd -d u3 groupe2
Removing user u3 from group groupe2

# groups u3
u3 : groupe2
```
*On remarque que le groupe n'a pas changé, cela est expliqué par le fait que u3 ne possèdant un seul groupe, il sera remis dans le groupe portant son nom. Cette réatribution sera effectuée lors de la prochaine connexion de u3.*

&nbsp;


**16. Modifiez le compte de u4 de sorte que :**
— **il expire au 1er juin 2020**
— **il faut changer de mot de passe avant 90 jours**
— **il faut attendre 5 jours pour modifier un mot de passe**
— **l’utilisateur est averti 14 jours avant l’expiration de son mot de passe**
— **le compte sera bloqué 30 jours après expiration du mot de passe**

```
# chage -E 2020-06-1 u4
# chage -M 90 u4
# chage -m 5 u4
# chage -W 14 u4
# chage -I 30 u4
```
`-E` change la date d'expiration de l'utilisateur
`-M` change la durée maximale de validité du mot de passe
`-m` change le délai avant de pouvoir rechanger son mot de passe
`-W` change le nombre de jour où l'utilisateur est averti avant l'expiration de son mot de passe
`-I` change le nombre de jours après lequel, suite à l'expiration de son mot de passe, l'utilisateur est bloqué

On vérifie les valeurs affectées
```
# chage -l u4
Last password change                                    : Mar 13, 2020
Password expires                                        : Jun 11, 2020
Password inactive                                       : Jul 11, 2020
Account expires                                         : Jun 01, 2020
Minimum number of days between password change          : 5
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 14
```
&nbsp;


**17. Quel est l’interpréteur de commandes (Shell) de l’utilisateur root ?**

Connecté en tant que root:

```
root@server:~# printenv SHELL
/bin/bash
```

&nbsp;


**18. à quoi correspond l’utilisateur nobody ?**

C'est un utilisateur existant par défaut, ayant les droits minimum possibles sur le système.
Il est utilisé pour tester des outils ne nécessitant aucun droits particuliers, ou pour des services sensibles (comme des serveurs webs, databases, ...)

&nbsp;


**19. Par défaut, combien de temps la commande sudo conserve-t-elle votre mot de passe en mémoire ? Quelle commande permet de forcer sudo à oublier votre mot de passe ?**

Par défaut, le temps d'expiration du mot de passe sudo est de 15 minutes.
On peut le modifier en se connectant sur l'utilisateur root, éxécutant la commande `visudo`
<pre>
Et modifier le fichier:
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset,<b><u>timestamp_timeout=0</u></b>
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:$

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification

</pre>
*On a ajouté `timestamp_timeout=0` pour désactiver la sauvegarde du mot de passe lors d'un sudo*
&nbsp;

***

&nbsp;

### Exercice 2. Gestion des permissions

**1. Dans votre $HOME, créez un dossier test, et dans ce dossier un fichier fichier contenant quelques lignes de texte. Quels sont les droits sur test et fichier ?**

```
$ mkdir test
$ echo 'aaa' > test/fichier
$ ls -l
drwxrwxr-x 2 herysia herysia 4096 Mar 13 17:07 test

$ ls -l test
-rw-rw-r-- 1 herysia herysia 4 Mar 13 17:08 fichier

```
On a donc 775 pour le dossier, et 664 pour le fichier

&nbsp;


**2. Retirez tous les droits sur ce fichier (même pour vous), puis essayez de le modifier et de l’afficher en tant que root. Conclusion ?**

```
# chmod 000 test/fichier
# ls -l test
total 4
---------- 1 herysia herysia 4 Mar 13 17:08 fichier

# echo 'bbb' > test/fichier
# cat test/fichier
bbb
# rm test/fichier
# ls -l test
total 0
```
On remarque que, en tant que root, on est tout de même capable de lire, écrire et supprimer le fichier malgrès le fait que toutes les permissions aient été retirées
&nbsp;


**3. Redonnez vous les droits en écriture et exécution sur fichier puis exécutez la commande echo "echo Hello" > fichier. On a vu lors des TP précédents que cette commande remplace le contenu d’un fichier s’il existe déjà. Que peut-on dire au sujet des droits ?**

```
$ sudo chmod u+wx fichier
$ ls -l
total 4
--wx------ 1 herysia herysia 4 Mar 13 17:08 fichier


$ echo "echo Hello" > fichier
$ ls -l
total 4
--wx------ 1 herysia herysia 11 Mar 13 17:32 fichier
```

*Les droites d'ecriture pour l'utilisateur sont suffisant car nous somme propriétaires du fichier*
&nbsp;


**4. Essayez d’exécuter le fichier. Est-ce que cela fonctionne ? Et avec sudo ? Expliquez**

```
$ ./fichier
bash: ./fichier: Permission denied
$ sudo ./fichier
Hello
```
Sans sudo, on ne peut pas exécuter le fichier, pourtant l'utilisateur possède les droits d'éxécution. C'est le cas car l'utilisateur n'a pas les droits de lecture.
Avec sudo, on passe par dessus les permissions, on peut donc l'exécuter sans problèmes

Correction:
```
$ sudo chmod u+r fichier
$ ./fichier
Hello
```

&nbsp;


**5. Placez-vous dans le répertoire test, et retirez-vous le droit en lecture pour ce répertoire. Listez le contenu du répertoire, puis exécutez ou affichez le contenu du fichier fichier. Qu’en déduisez-vous ? Rétablissez le droit en lecture sur test**

```
$ sudo chmod u-r test
$ ls test
ls: cannot open directory 'test': Permission denied

$ cat test/fichier
echo Hello

$ ./test/fichier
Hello
```
*On ne peut pas afficher le contenu du dossier test car on a pas les droits en lecture, néamoins les permissions du fichier n'ont pas changer, donc on peut toujours le lire ou l'exécuter*

On rétalit les droits
`$ sudo chmod u+r test`

&nbsp;


**6. Créez dans test un fichier nouveau ainsi qu’un répertoire sstest. Retirez au fichier nouveau et au répertoire test le droit en écriture. Tentez de modifier le fichier nouveau. Rétablissez ensuite le droit en écriture au répertoire test. Tentez de modifier le fichier nouveau, puis de le supprimer. Que pouvez-vous déduire de toutes ces manipulations ?**

```
$ echo 'test' > nouveau
$ mkdir sstest
$ sudo chmod u-w nouveau
$ cd ..
$ sudo chmod u-w test
$ echo 'test' > test/nouveau
bash: test/nouveau: Permission deniedrm 
'test' > test/nouveau
bash: test/nouveau: Permission denied

$ sudo chmod u+w test
$ echo 'test' > test/nouveau
-bash: test/nouveau: Permission denied
$ rm test/nouveau
rm: remove write-protected regular file 'test/nouveau'? yes
$ ls test
fichier  sstest
```
Retirer les droits d'écriture au dossier et au fichier nous empèche d'écrire ou de supprimer le fichier.
Remettre les droits au dossier (sans les remettre au fichier) nous empèche toujours d'écrire sur le fichier, néamoins nous pouvons écrire dans le dossier, et donc supprimer le fichier
&nbsp;


**7. Positionnez vous dans votre répertoire personnel, puis retirez le droit en exécution du répertoire test. Tentez de créer, supprimer, ou modifier un fichier dans le répertoire test, de vous y déplacer, d’en lister le contenu, etc…Qu’en déduisez vous quant au sens du droit en exécution pour les répertoires ?**

```
$ chmod u-x test

$ touch test/aa
touch: cannot touch 'test/aa': Permission denied

$ echo 'test' > test/fichier
bash: test/fichier: Permission denied


$ ls -l test
ls: cannot access 'test/fichier': Permission denied
ls: cannot access 'test/sstest': Permission denied
ls: cannot access 'test/nouveau': Permission denied
total 0
-????????? ? ? ? ?            ? fichier
-????????? ? ? ? ?            ? nouveau
d????????? ? ? ? ?            ? sstest


$ rm test/fichier
rm: cannot remove 'test/fichier': Permission denied

$ ./test/fichier
bash: ./test/fichier: Permission denied
```
Supprimer les droits d'éxécution du dossier nous empèche de créer, supprimer ou modifier un fichier.
Nous pouvons toujour lister le contenu, mais il y a des erreur indiquant qu'il est impossible d'accèder aux éléments, et donc d'en déduire leurs caractéristiques autre que leur nom

&nbsp;


**8. Rétablissez le droit en exécution du répertoire test. Positionnez vous dans ce répertoire et retirez lui à nouveau le droit d’exécution. Essayez de créer, supprimer et modifier un fichier dans le répertoire test, de vous déplacer dans ssrep, de lister son contenu. Qu’en concluez-vous quant à l’influence des droits que l’on possède sur le répertoire courant ? Peut-on retourner dans le répertoire parent avec ”cd ..” ? Pouvez-vous donner une explication ?**

```
$ chmod u+x test
$ cd test
$ chmod u-x ../test
$ touch a
touch: cannot touch 'a': Permission denied
$ ls
ls: cannot open directory '.': Permission denied
$ rm fichier
rm: cannot remove 'fichier': Permission denied
$ echo 'aaa' > fichier
bash: fichier: Permission denied
$ cd ssrep
bash: cd: ssrep: Permission denied
$ ls ssrep
ls: cannot access 'ssrep': Permission denied
```
On peut en conclure que nos droits sont calqués de nos permissions du répertoire courant.
Dans le répertoire test, nous n'avons pas les droits d'exécution, nous ne pouvons donc pas exécuter de commande (donc impossible de créer, supprimer, modifier un fichier, impossible de se déplacer ou liste le contenu d'un dossier ou du dossier courant)

`$ cd ..` Fonctionne pourtant car nous possédons les permissions dans le dossier parent. Cela n'etait pas le cas pour le dossier ssrep, malgrès le fait que nous ayons les permessions du ce dossier, n'ayant pas les permissions sur son parent il n'est pas possible d'y accéder

&nbsp;


**9. Rétablissez le droit en exécution du répertoire test. Attribuez au fichier fichier les droits suffisants pour qu’une autre personne de votre groupe puisse y accéder en lecture, mais pas en écriture.**

```
$ chmod u+x test
$ chmod g+r test/fichier
$ chmod g-w test/fichier
```

&nbsp;


**10. Définissez un umask très restrictif qui interdit à quiconque à part vous l’accès en lecture ou en écriture, ainsi que la traversée de vos répertoires. Testez sur un nouveau fichier et un nouveau répertoire.**

```
$ umask 077
$ umask -S
u=rwx,g=,o=
```
Testons de créer un nouveau dossier et fichier, et d'y acceder nous même
```
$ mkdir abc
$ echo 'test' > abc/aaa
$ cd abc
$ ls
aaa
$ cat aaa
test
```
Cela fonctionne

Changeons d'utilisateur et réessayons
```
$ su u1
Password:
$ cd /home/herysia
$ ls
aaa  abc  repo-cpe  scripts  test
$ cd abc
sh: 32: cd: can't cd to abc
$ ls abc
ls: cannot open directory 'abc': Permission denied
$ touch abc/test
touch: cannot touch 'abc/test': Permission denied
```
On observe que l'utilisateur ne peut pas acceder au dossier, lire un fichier, ni écrire dedans

*Note: On peut toujours accéder aux autres éléments tels que le dossier test car umask n'est appliqué uniquement aux nouveaux dossiers et fichiers*
&nbsp;


**11. Définissez un umask très permissif qui autorise tout le monde à lire vos fichiers et traverser vos répertoires, mais n’autorise que vous à écrire. Testez sur un nouveau fichier et un nouveau répertoire**

```
$ umask 022
$ mkdir q11
$ echo 'test q11' > q11/test
$ su u1

```
On teste avec un utilisateur
```
$ su u1
Password:
$ cd /home/herysia
$ ls
aaa  abc  q11  repo-cpe  scripts  test
$ cd q11
$ ls
test
$ cat test
test q11
$ mkdir u1
mkdir: cannot create directory ‘u1’: Permission denied
$ touch u1
touch: cannot touch 'u1': Permission denied
```
On vérifie que l'on peut bien naviguer dans le nouveau dossier créé, lire les fichiers, mais pas créer de dossier ou fichier
&nbsp;


**12. Définissez un umask équilibré qui vous autorise un accès complet et autorise un accès en lecture aux membres de votre groupe. Testez sur un nouveau fichier et un nouveau répertoire.**

```
$ umask 037
$ mkdir q12
$ echo 'test q12' > q12/test
```
On teste avec un utilisateur
```
$ cd /home/herysia
$ ls
aaa  abc  q11  q12  repo-cpe  scripts  test
$ cd q12
sh: 3: cd: can't cd to q12
```
L'utilisateur n'étant pas dans le groupe herysia, cela ne fonctionne pas.
Ajoutons le `$ sudo usermod -a -G herysia u1
`
et réessayons
```
$ su u1
Password:
$ cd /home/herysia
$ ls
aaa  abc  q11  q12  repo-cpe  scripts  test
$ cd q12
$ ls
test
$ cat test
test q12
$ mkdir u1
mkdir: cannot create directory ‘u1’: Permission denied
$ touch u1
touch: cannot touch 'u1': Permission denied
```
Ce même utilisateur, faisant désormais partie du groupe herysia, peut désormais naviguer dans les dossiers et lire les fichiers, mais toujour pas écrire des fichiers ou dossier
&nbsp;


**13. Transcrivez les commandes suivantes de la notation classique à la notation octale ou vice-versa (vous pourrez vous aider de la commande stat pour valider vos réponses) :**
- *chmod u=rx,g=wx,o=r fic*
`chmod 534 fic`
- *chmod uo+w,g-rx fic en sachant que les droits initiaux de fic sont r--r-x---*
`chmod 604 fic`
- *chmod 653 fic en sachant que les droits initiaux de fic sont 711*
`chmod u-x,g-r,o-w fic`
- *chmod u+x,g=w,o-r fic en sachant que les droits initiaux de fic sont r--r-x---*
`chmod 520 fic`


&nbsp;


**14. Affichez les droits sur le programme passwd. Que remarquez-vous ? En affichant les droits du fichier /etc/passwd, pouvez-vous justifier les permissions sur le programme passwd ?**

```
$ stat -c %A /etc/passwd
-rw-r--r--
```
Ces permissions correspondent aux permissions par défaut pour un fichier.
Le propriétaire de se fichier est root, donc les permissions users n'ont pas d'importance.
Les autres utilisateurs (group et other) possède seulement le droit en lecture, ce qui leur permet de lister les utilisateurs, connaître leur groupe, etc (comme vu précédemment). Mais ils ne peuvent pas l'éditer (en effet il est obligatoire d'être root pour créer un utilisateur)

&nbsp;


#### Pour les plus rapides :


**15. Access Control Lists (ACL) : suivez le tutoriel de cette page : https://doc.ubuntu-fr.org/acl.**


&nbsp;


**16. Quotas disques : suivez le tutoriel de cette page : https://doc.ubuntu-fr.org/quota.**


&nbsp;