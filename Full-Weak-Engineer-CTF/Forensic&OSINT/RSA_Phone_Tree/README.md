## RSA Phone Tree (124pt / 240 solves) [Medium]
> PendelとQuesoから電話がかかってきた…。これはセキュアな電話なので、電話番号を入力するのも一苦労だ。
> 
> Pendel and Queso called me... This line is so secure that just dialing the number feels like a challenge.
> 
> 添付ファイル: RSA_Phone_Tree.zip

zipファイルの中には以下のファイルが入っていた。
- challenge.py
- message.wav
- p_dial.wav
- q_dial.wav

challenge.pyの内容を次に示す。
```python
import numpy as np
from scipy.io.wavfile import write
from Crypto.Util.number import getPrime, bytes_to_long
import random

dtmf_freqs = {
    '1': (697, 1209), '2': (697, 1336), '3': (697, 1477), 'A': (697, 1633),
    '4': (770, 1209), '5': (770, 1336), '6': (770, 1477), 'B': (770, 1633),
    '7': (852, 1209), '8': (852, 1336), '9': (852, 1477), 'C': (852, 1633),
    '*': (941, 1209), '0': (941, 1336), '#': (941, 1477), 'D': (941, 1633),
}

def text_to_tone(text, filename, fs=8000, tone_time=0.08, silence_time=0.10):
    samples = []
    for char in text:
        if char not in dtmf_freqs:
            continue
        f1, f2 = dtmf_freqs[char]
        t = np.linspace(0, tone_time, int(fs * tone_time), endpoint=False)
        
        phase1 = random.uniform(0, 2 * np.pi)
        phase2 = random.uniform(0, 2 * np.pi)
        tone = 0.5 * np.sin(2 * np.pi * f1 * t + phase1) + 0.5 * np.sin(2 * np.pi * f2 * t + phase2)
        samples.append(tone)
        samples.append(np.zeros(int(fs * silence_time)))
    sig = np.concatenate(samples)
    sig = (sig * 32767).astype(np.int16)
    write(filename, fs, sig)

p = getPrime(512)
q = getPrime(512)
n = p * q
e = 65537

flag = b"dummy{What is Your Telephone Number? *^_^* }"
m = bytes_to_long(flag)
c = pow(m, e, n)

p_str = str(p)
q_str = str(q)
c_str = str(c)

text_to_tone(p_str, "p_dial.wav")
text_to_tone(q_str, "q_dial.wav")
text_to_tone(c_str, "message.wav")
```

wavファイルはいずれも「ピポパ」といった擬音で表現されるところの、電話でボタンを押した時に出る音が記録されていた。これはDTMFというらしい。pyファイルの`text_to_tone()`が何をしているのかはよく分からなかったが、要はテキストをDTMFに変換し、wavファイルとして書き出す関数なのだと推測した。となると、DTMFをテキストに逆変換する必要がある。"DTMF Decoder"で検索するとドンピシャな[ツール](https://dtmf.netlify.app/)が見つかった。ありがたいことである。これにより`p`, `q`の値と暗号文が判明、後はRSA暗号を解くだけなので、次に示すソルバを書いた。
```python
from Crypto.Util.number import *

p = 10983959977181906221944070277050249695159719065797979860198327906342517841465796251340510314543825459306549312379388467919085481804275635828255424368896277
q = 8678036199170488884780093394192150083464607667624372136417058065258513864387020799930756914928144817991697023271050010755986626888722989975295810617844267
n = p * q
e = 65537
c = 70508688516557799681651812792642651067871903715364993117923211379841452025135849618182868501557625870789695156447009505178161039560403696520152794746543040447501478286428695065976528304147698191471798860041657166559094242292315109255014028450790613496046735617040355184531712588116133880923988426223187516216

phi = (p-1) * (q-1)
d = pow(e, -1, phi)
m = pow(c, d, n)

print(long_to_bytes(m).decode())
```

実行するとフラグが得られた。

### `fwectf{Y0ur_7e13ph0n3_Num83r_15_6700_WoW!__H3110?}`

ﾘﾝﾘﾝﾘﾘﾝﾘﾝﾘﾝﾘﾘﾝﾘﾝ......