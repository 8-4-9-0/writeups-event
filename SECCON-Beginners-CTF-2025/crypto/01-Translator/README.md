## 01-Translator (100pt / 280 solves) [easy]
> バイナリ列は読めない？じゃあ翻訳してあげるよ！  
> `nc [問題鯖のホスト名] [ポート番号]`

次に示すソースコードが与えられている。
```python
import os
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Util.number import bytes_to_long


def encrypt(plaintext, key):
    cipher = AES.new(key, AES.MODE_ECB)
    return cipher.encrypt(pad(plaintext.encode(), 16))

flag = os.environ.get("FLAG", "CTF{dummy_flag}")
flag_bin = f"{bytes_to_long(flag.encode()):b}"
print(flag_bin)
trans_0 = input("translations for 0> ")
trans_1 = input("translations for 1> ")
flag_translated = flag_bin.translate(str.maketrans({"0": trans_0, "1": trans_1}))
print(flag_translated)
key = os.urandom(16)
print("ct:", encrypt(flag_translated, key).hex())
```
フラグ文字列を一旦バイナリに変換した後、`0`と`1`をそれぞれユーザが入力した文字列に置き換え、AESのECBモードで暗号化したものを出力する、という内容になっている。
暗号化に用いる鍵は乱数となっており、これを知ることは難しそうである。  
ここで、AESのECBモードは16バイトのデータブロックごとに同じ鍵を使い回すことを利用する。つまり、`0`と`1`を適当な16バイトの文字列にtranslateしてやれば、暗号化された16バイトの文字列がやはり`0`と`1`に対応するため、
鍵を求めることなく復号できる。  
フラグ文字列の頭文字は`c`で変わらない。`c`を2進数で表すと`1100011`であるから、暗号化文字列の0～15バイト目が`1`、32～47バイト目が`0`に対応する。  
以上より、次に示すソルバを書いた。
```python
from pwn import *
from Crypto.Util.number import *

io = remote(hostname, port)

io.sendafter('translations for 0> ', 'ABCDEFGHIJKLMNOP\n')
io.sendafter('translations for 1> ', 'abcdefghijklmnop\n')

io.recvuntil('ct: ')
encrypted = io.recvline().decode()
zero = encrypted[64:96]
one = encrypted[0:32]
flag_bin = encrypted.replace(zero, '0').replace(one, '1')
flag = int(flag_bin[:-33], 2)
print(long_to_bytes(flag).decode())
```
`flag_bin`の後ろ16バイトを削っているのは、この部分は暗号化のためのパディングとなっており、関係のない部分だからである。
このソルバを実行するとフラグが得られた。
```
python3 solver.py
[+] Opening connection to [URL] on port [PORT]: Done
solver.py:6: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  io.sendafter('translations for 0> ', 'ABCDEFGHIJKLMNOP\n')
tube.py:866: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  res = self.recvuntil(delim, timeout=timeout)
solver.py:7: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  io.sendafter('translations for 1> ', 'abcdefghijklmnop\n')
solver.py:9: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  io.recvuntil('ct: ')

ctf4b{n0w_y0u'r3_4_b1n4r13n}
[*] Closed connection to [URL] on port [PORT]
```

### `ctf4b{n0w_y0u'r3_4_b1n4r13n}`
