## wasm_S_exp (100pt / 330 solves)
> フラグをチェックしてくれるプログラム

次に示すWebAssemblyコードが与えられる。（長いため少し省略している）
```wasm
(module
  (memory (export "memory") 1 )
  (func (export "check_flag") (result i32)
    i32.const 0x7b
    i32.const 38
    call $stir
    i32.load8_u
    i32.ne
    if
      i32.const 0
      return
    end

    i32.const 0x67
    i32.const 20
    call $stir
    i32.load8_u
    i32.ne
    if
      i32.const 0
      return
    end

    ...

    i32.const 0x41
    i32.const 26
    call $stir
    i32.load8_u
    i32.ne
    if
      i32.const 0
      return
    end

    i32.const 1
    return
  )

  (func $stir (param $x i32) (result i32)
    i32.const 1024
    i32.const 23
    i32.const 37
    local.get $x
    i32.const 0x5a5a
    i32.xor
    i32.mul
    i32.add
    i32.const 101
    i32.rem_u
    i32.add
    return
  )
)
```

ぱっと見では、このコードも入力データをフラグ文字列と比較させているように見える。が、`$stir`周りが何を行っているのか分からない。  
ChatGPTに訊いてみると、`$stir`は`(((実引数) ^ 0x5a5a) * 37 + 23) % 101 + 1024`の結果を返す関数だという。  
また、`i32.load8_u`もよく分からなかったので訊いてみると、「スタックの最上位に積まれた値をアドレスとし、メモリ上のそのアドレスに格納されている8ビットのデータを読み取ってスタックに積む」命令とのこと。  
つまり、アドレスが小さい順、すなわち`$stir`の結果が小さい順に比較対象のASCIIコードを並べればフラグになりそうである。  
ということで、次に示すソルバを書いた。これはWebAssemblyコードからASCIIコードと対応するメモリアドレスを抜き出してそれらで構成される2次元配列を作成し、メモリアドレスについて昇順ソートを行った後、配列の最初からASCIIコードを文字列に変換して一文字ずつ出力するPythonスクリプトである。
```python
file = 'check_flag.wat'
i = 1
chars = []
item = [0, 0]

with open(file, mode='r') as f:
    for line in f:
        if i % 10 == 4 and i <= 249:
            item[0] = int(line[line.find('t')+2:line.find('t')+6], 0)
        elif i % 10 == 5 and i <= 249:
            item[1] = int(line[line.find('t')+2:line.find('t')+5].strip('\n'), 10)
            item[1] = 1024 + ((23 + 37 * (item[1] ^ 0x5a5a)) % 101)
            chars.append(item.copy())
        i += 1
    chars_sorted = sorted(chars, key=lambda x: x[1])
    print(chars_sorted)
    for c in chars_sorted:
        print(chr(c[0]), end="")
    print()
```

このソルバを実行することでフラグが得られた。

### `ctf4b{WAT_4n_345y_l0g1c!}`
