## No need Logical Thinking (132pt / 219 solves) [Easy]
> 論理的思考はどんな物事にも必要。
> 
> Logical Thiniking is need for everything.
>
> 添付ファイル: No_need_Logical_Thinking.zip

次に示すソースコードと、その出力が`output.txt`として与えられている。
```Prolog
process_flag(FileName) :-
    open(FileName, read, Stream),           
    read_string(Stream, _, Content),        
    close(Stream),                          
    string_codes(Content, Codes),           
    transform_codes(Codes, 1, Transformed),
    string_codes(NewString, Transformed),   
    writeln(NewString).                     


transform_codes([], _, []).
transform_codes([H|T], Index, [NewH|NewT]) :-
    NewH is H + Index,                      
    NextIndex is Index + 1,                  
    transform_codes(T, NextIndex, NewT).     


%EXECUTE
%?- process_flag('flag.txt').
```

拡張子は`.pl`だったが、LLMに訊いてみるとこれはPerlではなくPrologのコードだという。気になるのは`transform_codes()`の部分で、この関数はリスト内の要素について、前から1つずつ引数`Index`を足していくという処理を行う。なお、`Index`も加算処理の合間に1ずつプラスされていく。要は、フラグが欲しければこれと逆のことを`output.txt`に行えば良い。渡されている`Index`の初期値は`1`であることに注意してソルバを書くと、以下の通りになった。
```python
with open('output.txt', 'r') as f:
    flag = f.read()

i = 1
for c in flag:
    print(chr(ord(c)-i), end="")
    i += 1

print()
```

実行すると、フラグが得られた。

### `fwectf{the_Pr010g_10gica1_Languag3!}`