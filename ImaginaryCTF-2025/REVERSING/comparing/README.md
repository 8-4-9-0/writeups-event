## comparing (100pt / 558 solves)
> I put my flag into this program, but now I lost the flag. Here is the program, and the output. Could you use it to find the flag?
> 
> 添付ファイル: comparing.cpp, output.txt

comparing.cppの内容は以下。なお最初に大量の`#include`があったが、省略している。
```cpp
class Compare {
public:
    bool operator()(tuple<char, char, int> a, tuple<char, char, int> b) {
        return static_cast<int>(get<0>(a)) + static_cast<int>(get<1>(a)) > static_cast<int>(get<0>(b)) + static_cast<int>(get<1>(b));
    }
};

string even(int val1, int val3, int ii) {
    string out = to_string(val1) + to_string(val3) + to_string(ii);
    string x = to_string(val1) + to_string(val3);
    for (int i = x.size() - 1; i >= 0; i--) {
        out += x[i];
    }
    return out;
}

string odd(int val1, int val3, int ii) {
    int out = stoi(to_string(val1) + to_string(val3) + to_string(ii));
    int i = 0;
    int addend = 0;
    while (i < 100) { addend += i; i++; }
    i--;
    while (i >= 0) { addend -= i; i--; }
    return to_string(out + addend);
}

int main()
{
    string flag = "REDACTED";
    priority_queue<tuple<char, char, int>, vector<tuple<char, char, int>>, Compare> pq;
    for (int i = 0; i < flag.size() / 2; i++) {
        tuple<char, char, int> x = { flag[i * 2],flag[i * 2 + 1],i };
        pq.push(x);
    }
    vector<string> out;
    while (!pq.empty()) {
        int val1 = static_cast<int>(get<0>(pq.top()));
        int val2 = static_cast<int>(get<1>(pq.top()));
        int i1 = get<2>(pq.top());
        pq.pop();
        int val3 = static_cast<int>(get<0>(pq.top()));
        int val4 = static_cast<int>(get<1>(pq.top()));
        int i2 = get<2>(pq.top());
        pq.pop();
        if (i1 % 2 == 0) { out.push_back(even(val1, val3, i1)); }
        else { out.push_back(odd(val1, val3, i1)); }
        if (i2 % 2 == 0) { out.push_back(even(val2, val4, i2)); }
        else { out.push_back(odd(val2, val4, i2)); }
    }
    for (int i = 0; i < out.size(); i++) {
        cout << out[i] << endl;
    }
}
```

output.txtの内容は以下。
```
9548128459
491095
1014813
561097
10211614611201
5748108475
1171123
516484615
114959
649969946
1051160611501
991021
1231012101321
9912515
11411511
1151164611511
```

comparing.cppが行っている処理は少しややこしいため、LLMに訊いたりググったりして得た情報を参考に内容を説明していく。（C++はあまり触ったことがないので、もし間違いがあったら教えてください）
- `Compare`クラス  
    `char`値2つと`int`値1つで構成されたタプル`a`, `b`を引数に取る、後の`priority_queue`に渡すためのカスタム比較関数オブジェクトを持っている。その働きとしては、`a`, `b`のそれぞれについて2つの`char`値（を`int`にキャストしたもの）の和を比較し、実際に`a > b`すなわち`true`であれば`b`の優先度を高く、逆に`a <= b`すなわち`false`であれば`a`の優先度を高くする、というものである。
- `even()`  
    `to_string()`は`int`型の数値をそのまま`char`型の文字列に変換する関数。for文では文字列`x`を後ろから1文字ずつ`out`に結合している。つまり最終的な戻り値`out`の中身は
    ```
    to_string(val1) + to_string(val3) + to_string(ii) + to_string(val3)(逆順) + to_string(val1)(逆順)
    ```
    となる。具体的にはこんな感じ。
    ```
    even(102, 110, 5); // -> '1021105011201'
    ```
- `odd()`  
    `stoi()`は`char`型の文字列になっている数字を`int`型に変換する関数。この変換は後々`addend`を足すために行っているわけだが、2つのwhile文で操作されている`addend`の値は最終的に`0`に戻ってくるため、実は何の意味もなさない。よって戻り値の中身は
    ```
    to_string(val1) + to_string(val3) + to_string(ii)
    ```
    という単純なものになる。具体的にはこんな感じ。

    ```
    odd(102, 110, 5); // -> '1021105'
    ```
- `main()`  
    まず最初に優先度付きキュー（`priority_queue`）である`pq`を用意し、そこにタプルを入れていく。入れられるタプルについては、`flag`を2文字ずつの組に分け、前の組から`<1文字目, 2文字目, 組の通し番号>`という形で作られる。また、上述した`Compare`により、1文字目と2文字目のASCIIの和が小さい順にタプルが並ぶようになっている。  
    キューにタプルを詰め終えたら、while文にてメインの処理を行う。流れとしては以下のような感じになる。
    1. キューの先頭から2つタプルを取り出し、1つ目の各要素を`val1`, `val2`, `i1`に、2つ目の各要素を`val3`, `val4`, `i2`に格納する。
    2. `i1`が偶数ならば`even(val1, val3, i1)`、奇数ならば`odd(val1, val3, i1)`を実行し、その結果を`out`の末尾に追加する。
    3. `i2`が偶数ならば`even(val2, val4, i2)`、奇数ならば`odd(val2, val4, i2)`を実行し、その結果を`out`の末尾に追加する。
    
    最後のfor文は`out`の内容を出力しているだけ。キューに詰め込んだ時点でもうある程度シャッフルされているというのに、`val1`～`val4`の順番まで入れ替えるという抜かりの無さである。

これで一通りの説明はおしまい。といってもこれだけでは理解しづらいだろうから、`ictf{dummy_flag}`という偽フラグを使って具体的な処理の流れを追ってみる。
```
ictf{dummy_flag}
```
↓ タプル化
```
{'i', 'c', 0}, {'t', 'f', 1}, {'{', 'd', 2}, {'u', 'm', 3}, {'m', 'y', 4}, {'_', 'f', 5}, {'l', 'a', 6}, {'g', '}', 7}
```
↓ `Compare`による並び替え
```
{'_', 'f', 5}, {'i', 'c', 0}, {'l', 'a', 6}, {'t', 'f', 1}, {'{', 'd', 2}, {'u', 'm', 3}, {'g', '}', 7}, {'m', 'y', 4}, 
```
↓ while文内での処理
```
odd('_', 'i', 5)
even('f', 'c', 0)
even('l', 't', 6)
odd('a', 'f', 1)
even('{', 'u', 2)
odd('d', 'm', 3)
odd('g', 'm', 7)
even('}', 'y', 4)
```
↓ 結果
```
951055
10299099201
1081166611801
971021
1231172711321
1001093
1031097
1251215121521
```

さて、これで処理の流れを分解することができたので、この流れを逆に行うソルバを作りたい。最終的にはLLMに土台を用意してもらい、以下のソルバをこしらえた。
```python
def reconstruct_flag(lists):
    i = 0
    for val1, val2, idx in lists:
        if i % 2 == 0:
            temp = lists[i][1]
            lists[i][1] = lists[i+1][0]
            lists[i+1][0] = temp
        i += 1

    lists_sorted = sorted(lists, key=lambda x: x[2])
    flag_chars = ['0'] * 2 * 16
    for val1, val2, idx in lists_sorted:
        flag_chars[idx*2] = chr(val1)
        flag_chars[idx*2+1] = chr(val2)
    
    return ''.join(flag_chars)

# --- 使い方の例 ---
if __name__ == "__main__":
    # ここに元に戻したタプルのリストを入れる
    lists = [
        [95,48,12],
        [49,109,5],
        [101,48,13],
        [56,109,7],
        [102,116,14],
        [57,48,10],
        [117,112,3],
        [51,64,8],
        [114,95,9],
        [64,99,6],
        [105,116,0],
        [99,102,1],
        [123,101,2],
        [99,125,15],
        [114,115,11],
        [115,116,4]
    ]
    flag = reconstruct_flag(lists)
    print("Recovered flag:", flag)
```

`lists`はoutput.txtの中身を自分で分解して落とし込んだものである。このソルバを実行するとフラグが得られた。
```
$ python3 solver.py 
Recovered flag: ictf{cu3st0m_c0mp@r@t0rs_1e8f9e}
```

### `ictf{cu3st0m_c0mp@r@t0rs_1e8f9e}`