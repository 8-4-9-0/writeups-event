## url-checker (100pt / 606 solves) [easy]
> 有効なURLを作れますか？  
> nc [問題鯖のホスト名] [ポート番号]

次に示すソースコードが与えられている。
```python
from urllib.parse import urlparse

print(
    r"""
 _   _ ____  _        ____ _               _             
| | | |  _ \| |      / ___| |__   ___  ___| | _____ _ __ 
| | | | |_) | |     | |   | '_ \ / _ \/ __| |/ / _ \ '__|
| |_| |  _ <| |___  | |___| | | |  __/ (__|   <  __/ |   
 \___/|_| \_\_____|  \____|_| |_|\___|\___|_|\_\___|_|   

allowed_hostname = "example.com"                                                         
>> """,
    end="",
)

allowed_hostname = "example.com"
user_input = input("Enter a URL: ").strip()
parsed = urlparse(user_input)

try:
    if parsed.hostname == allowed_hostname:
        print("You entered the allowed URL :)")
    elif parsed.hostname and parsed.hostname.startswith(allowed_hostname):
        print(f"Valid URL :)")
        print("Flag: ctf4b{dummy_flag}")
    else:
        print(f"Invalid URL x_x, expected hostname {allowed_hostname}, got {parsed.hostname if parsed.hostname else 'None'}")
except Exception as e:
    print("Error happened")
```
入力したURLを[urlparse関数](https://docs.python.org/ja/3.13/library/urllib.parse.html)でパースし、`hostname`が`example.com`と完全一致はしないが、`example.com`から始まるものであればフラグが得られるというものであった。  
つまり、例えば`hostname`が`example.comm`となるようにURLを入力してやればフラグが得られそうである。  
実際にそのように入力すると、フラグが得られた。
```
$ nc [hostname] [port]

 _   _ ____  _        ____ _               _
| | | |  _ \| |      / ___| |__   ___  ___| | _____ _ __
| | | | |_) | |     | |   | '_ \ / _ \/ __| |/ / _ \ '__|
| |_| |  _ <| |___  | |___| | | |  __/ (__|   <  __/ |
 \___/|_| \_\_____|  \____|_| |_|\___|\___|_|\_\___|_|

allowed_hostname = "example.com"
>> Enter a URL: https://example.comm
Valid URL :)
Flag: ctf4b{574r75w17h_50m371m35_n07_53cur37}
```

### `ctf4b{574r75w17h_50m371m35_n07_53cur37}`
