2016.12.15 thu 14:00-15:50, 参加者9名(リモート含む) 議事メモ

# QUICトランスポート機能に関して tcpm/mptcp wg chair の西田先生にいろいろ聞いてみる会

https://github.com/shigeki/ask_nishida_about_quic_jp

# 1.概要(木村)

- IETF97でQUIC WG設立を受けてQUICに関する情報交換を行う。
- QUICにはTLSとの関係やトランスポートの話題がある。
- 今回はトランスポート。

# 2. IETF QUIC WG 全般について(Goal, Focus Area)(大津さん)

QUIC WG報告

https://speakerdeck.com/shigeki/quic-wgbao-gao

# 3. ディスカッション

## 3.1 QUIC vs TCP

### QUICの優れている点、ほか
- マルチストリームに対応している
- 一本のコネクションで複数のコンテンツを転送できる。
- Webを意識したデザイン。
- HTTP/2のマッピングからきたのでは。DNS over QUICも。
- コネクションマイグレーション
- IPアドレスが変わっても転送を継続できる。セッションのレジュームなど。
- (Q)Chromeで実行されている？ (A)Yes。プロトコルフォーマットに入っているので。実際の動きは確認されていない。

### ユーザランドかどうか
- QUICを使うアプリケーションのリコンパイルが必要になってしまう。
- WebアプリケーションではGoogleではデプロイしやすいが、、
- カーネル実装ではユーザのアプリケーション実装でよい。

### TCPの方が、QUICに対して優れている点
- ユーザランドのプログラムの方がQUICに対応しやすい。
- ハードウェア最適化の機能が使える。
- 通信相手が同じ時に異なるコネクションを張れるときにWindowサイズなどパラメーターを共有できる？
- QUICではバージョンネゴシエーションができる。新たな機能が入れやすい。
- 仮想化環境では応用可能性も。
- TCPとUDPの差、NAT expire が UDPの方が圧倒的に早い。30秒。TCPは4分。
- Googleでは1分を超えると減っていく。
- TLS1.3が入ることは色々解決するといわれる。
- HTTP/2のforkして新しく作るのではという主張もある。
- APIの視点ではシステムコールだけで実装できる。
- OSの影響範囲とアプリケーションの影響範囲の話
- performance enhanced proxy (代わりに再送するなど)

### QUICでも解決できないTCPの問題点はなんだろうか？
- STCPにあるもの：partial relyability
- 上位レイヤーとの関連
- 順序どおりに届いて欲しい場合は同じストリームに
- 今後OpenQUICライブラリが出てきたてきたときのAPIは気になる。
- IESGのdecisionだが基本的にIETFではAPIの標準化は行わない。
- Javaのようにバージョンアップしないと使えないことがあるのｋｓ？
- TCPでは互換性を維持しないといけない。
- QUICのバージョンが古くて通信できなくなることがある。
- バージョンアップすると使えるような過去を切り捨てる実装がありえるのか。
- 想定されている運用ではTCPにフォールバックする(大津)。
- RFCにしないとshapingなどの問題に対応できないのでは。ECN問題のように。

# 2. TCPM/MPTCP/関連

> QUICに導入されるトランスポート機能で長期間ドラフトままだったり、Expireしてしまたりしているものがあるのはなぜか？ ( TCP Cubic: 2008年論文発表、Linux 2.6.13より実装,TLP:ID-00 2012/07, expire 2013/08)

- 他にもexpireしていたものもあった。Cubicの位置づけはWGコンセンサスではinformationalRFCになる見込み。
- マイクロソフトの人が忙しかったり著者の反応が遅いなど。技術的な問題はない。
- Linuxに書かれているものをドキュメント化するにしても進行が遅い。
- New Renoを最初に行い、追々 Cubicは追々やっていく印象。
- Tail LossProbe(TLP)は提案されたが途絶えてしまった。著者がIETFに関わらなくなってしまったため。
- FACKも標準化しなければならず、元々の著者は標準化したいと思っていないことも要因。
- TLPはRACKの一アルゴリズムとして復活。

> 仕様化まで比較的時間がかかるなら、RACKとか新しいものは QUICの仕様化のタイミングにも間に合うのか？

- ソウルでは(次の質問の答えになってしまうが)、QUICはTCPと違うので New Reno を実装するのは不可能、スピリッツをQUICに入れるといったことがいわれていた。
- TCPのRACKとQUICのRACKも違う。TCPを参考にLoss detectionを作っていくのでは。
- TCPのアルゴリズムのForkを警戒している人はいた。

> Google版QUICはBBRをサポートしている。BBRはICCRGで検討中。今後の見込みは？

- BBRはChromiumに使われているが、標準化されるQUICに入るかどうかは分からない。
- LarsもIETFのQUICとしてドライブするようだ。
- BBRはメルマガにも書いたとおり実験のようだ。
- 次回のInterimは東京で行われる模様。
- 通信相手によってBBRかどうか、といったことがあるのか。膨大なデータを集められるGoogleはできる立場にあると思われる。

> TCP上の mutlpath は Seqの書き換えなどやはりミドルボックスの影響が大きく、通らないのか。(どの程度） (Connection IDベースのQUICは、network切り替えに対して比較的強いし、QUICは素早く暗号化を行うためミドルボックスの影響を受けにくい)

- Multipath TCPはミドルボックスの挙動を解析して作られたのである程度耐性がある。
- シーケンス番号が変わっても大丈夫。知らないタイプのオプションを削ってしまう
- ミドルボックスはありうる。

> QUICのMultipathのモティベーションは、、

- Multipathにしても仕様はあまり変わらないような気はする。
- Load sharingをadaptiveにできることはできるか。

## 3. QUIC の Transport 機能について

> QUICは、独自にトランスポート機能を開発するわけではなく、tcpm wgの成果をQUIC仕様に
> 取り込むのが大前提。ただしQUICは向けにアレンジしないといけない部分が出てくるだろう。
> どんな部分を変えないといけないのであろうか？

- TCPとはLoss recoveryが違う。New Renoはackベースのloss recoveryだが。
- タイムアウトしないのでROTしない、3782->6675もdup ackのloss recoveryも実装できない。
- Hybrid Slow Startは論文しかない。
- これらはこのまま入れることはないだろうと思う。
- RACKがコアとなって、、どうlossを見つけるかだけでしかない。
- 次にどのパケットを送ってリカバリーするのか、New Renoでは計算をしながら送るが、RACKを使うとどうなるかは未定なところがある。
- PRRはLoss recoveryの一つ。再送用のパケットが一気に出てしまうことがある。滑らかに新しいパケットを送ってバーストせずにリカバリーするためのもの。

# 4. 外部の方からの質問事項

3点、おさらい。

## 4.1 Happy Eyeball
https://github.com/shigeki/ask_nishida_about_quic_jp/issues/1

## 4.2 MTU
https://github.com/shigeki/ask_nishida_about_quic_jp/issues/2

## 4.3 Distinction between QUIC and DDoS
https://github.com/shigeki/ask_nishida_about_quic_jp/issues/3