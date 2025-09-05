## Poison Apple (100pt / 444 solves) [Beginner]
> iOSではウォッチドッグタイマが故障した時に返ってくる不思議な4バイトがあるらしい… 大文字にしてfwectf{}で囲ってね 例:1234ABCD→fwectf{1234ABCD}
> 
> Apparently, there are four special bytes returned when the watchdog timer fails on iOS... Capitalize them and enclose them in fwectf{}. e.g.1234ABCD→fwectf{1234ABCD}

試しに"iOS Watchdog"で検索すると[zennの記事](https://zenn.dev/numatech/articles/71f8fffc357c0e)がヒットし、こちらに載っていた。

### `fwectf{8BADF00D}`