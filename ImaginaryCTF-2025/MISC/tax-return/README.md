## tax-return (100pt / 193 solves)
> Here is a helpful ai tax return assistant. When I made it, I put a super secret flag into the system prompt. You will never find it! [問題サイトのURL]

問題サイトの見た目はこんな感じ。

<img src="images/image01.png" width="80%"></img>

確定申告アシスタントらしいのだが、PDFに書いてある文章に対して普通に返答を返してくれる。

<img src="images/image02.png" width="80%"></img>

（無題のドキュメント.pdfには"Hello"と書いてあるだけ）

早い話がプロンプトインジェクションの問題である。色々試行錯誤した結果、次のプロンプトでフラグを吐き出してくれた。（英語自体はDeepLで生成）

<img src="images/image03.png" width="80%"></img>

<img src="images/image04.png" width="70%"></img>

### `ictf{h0w_d1d_y0u_tr1ck_my_a1_@ss1st@nt?}`