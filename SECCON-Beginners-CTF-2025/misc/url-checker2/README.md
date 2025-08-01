## url-checker2 (100pt / 524 solves)
> 有効なURLを作れますか？ Part2  
> nc [問題鯖のホスト名] [ポート番号]

url-checkerと同様にソースコードが与えられており、フラグを表示するための条件がやや変更されている。
```python
from urllib.parse import urlparse

print(
    r"""
 _   _ ____  _        ____ _               _            ____  
| | | |  _ \| |      / ___| |__   ___  ___| | _____ _ _|___ \ 
| | | | |_) | |     | |   | '_ \ / _ \/ __| |/ / _ \ '__|__) |
| |_| |  _ <| |___  | |___| | | |  __/ (__|   <  __/ |  / __/ 
 \___/|_| \_\_____|  \____|_| |_|\___|\___|_|\_\___|_| |_____|
                                                              
allowed_hostname = "example.com"                                                         
>> """,
    end="",
)

allowed_hostname = "example.com"
user_input = input("Enter a URL: ").strip()
parsed = urlparse(user_input)

# Remove port if present
input_hostname = None
if ':' in parsed.netloc:
    input_hostname = parsed.netloc.split(':')[0]

try:
    if parsed.hostname == allowed_hostname:
        print("You entered the allowed URL :)")
    elif input_hostname and input_hostname == allowed_hostname and parsed.hostname and parsed.hostname.startswith(allowed_hostname):
        print(f"Valid URL :)")
        print("Flag: ctf4b{dummy_flag}")
    else:
        print(f"Invalid URL x_x, expected hostname {allowed_hostname}, got {parsed.hostname if parsed.hostname else 'None'}")
except Exception as e:
    print("Error happened")
```
条件を整理すると
1. `input_hostname`すなわち`netloc`が`:`を含んでおり、そこからホスト名単体を抜き出した部分が`example.com`と完全一致する
1. `hostname`が存在し、`example.com`と完全一致はしないが、`example.com`から始まる

この2つが満たされるとき、フラグが表示される。  
`hostname` = `netloc`からホスト名単体を抜き出した部分 なので、条件1.を成立させながら2.も成立させる方法がどうにも思い浮かばない。そこでChatGPTに聞いてみたところ、`https://example.com:80@exapmle.com.evil.com`なるURLを提案された。

```
$ nc [hostname] [port]

 _   _ ____  _        ____ _               _            ____
| | | |  _ \| |      / ___| |__   ___  ___| | _____ _ _|___ \
| | | | |_) | |     | |   | '_ \ / _ \/ __| |/ / _ \ '__|__) |
| |_| |  _ <| |___  | |___| | | |  __/ (__|   <  __/ |  / __/
 \___/|_| \_\_____|  \____|_| |_|\___|\___|_|\_\___|_| |_____|

allowed_hostname = "example.com"
>> Enter a URL: https://example.com:80@example.com.evil.com
Valid URL :)
Flag: ctf4b{cu570m_pr0c3551n6_0f_url5_15_d4n63r0u5}
```

### `ctf4b{cu570m_pr0c3551n6_0f_url5_15_d4n63r0u5}`
