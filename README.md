# QUICトランスポート機能に関して tcpm/mptcp wg chair の西田先生にいろいろ聞いてみる会

日時： 12/15(木) 14:00-15:30(JST)==12/14(水) 21:00-22:30(PST)

場所： JPNIC会議室

参加者： 西田先生(ハングアウト)、JPNIC木村さん、大津、村山さん(ハングアウト)

参加希望者は JPNIC 木村さんまで。

## IETF QUIC WG 全般について(Goal, Focus Area)

https://speakerdeck.com/shigeki/quic-wgbao-gao

## 参考ドラフト

QUIC Loss Detection and Congestion Control https://datatracker.ietf.org/doc/draft-ietf-quic-recovery/

## アジェンダ案

他に西田先生にQUICのトランスポート機能について聞いてみたい質問があればこのレポジトリへの issue や Pull Request をお願いします。

### 1.  QUIC vs TCP
QUICの優れている点
- TCPは kernel 実装、QUICはユーザランド実装。QUICは OS のバージョンアップが必要なく、速くdeployが可能
- QUICは packet numberが単調増加なのでLoss Detectionが簡単。TCPは再送で同じSeqを使うのでLoss Detectionが複雑になる。
- QUICはSACKが255 rangeも持てる、TCPは3 rangeしかない。
- QUICはハンドシェイクでCongestion Controlのアルゴリズムを選べる。TCPはカーネルパラメータ変更が必要。他のアプリも影響受ける。
- QUICはTLS1.3の暗号化がデフォルトで入り、すぐ暗号化通信を行う。TCPではtcpcyptが作業中だが…

他になにかあるだろうか？

TCPの方が、QUICに対して優れている点は？

QUICでも解決できないTCPの問題点はなんだろうか？

### 2. TCPM/MPTCP/関連
- QUICに導入されるトランスポート機能で長期間ドラフトままだったり、Expireしてしまったりしているものがあるのはなぜか？ ( TCP Cubic: 2008年論文発表、Linux 2.6.13 より実装,TLP:ID-00 2012/07, expire 2013/08)
- 仕様化まで比較的時間がかかるなら、RACKとか新しいものは QUIC の仕様化のタイミングにも間に合うのか？
- Google版QUICはBBRをサポートしている。BBRはICCRGで検討中。今後の見込みは？
- TCP上の mutlpath は Seq の書き換えなどやはりミドルボックスの影響が大きく、通らないのか。(どの程度）
   (Connection IDベースのQUICは、network切り替えに対して比較的強いし、QUICは素早く暗号化を行うためミドルボックスの影響を受けにくい)

### 3. QUIC の Transport 機能について

QUICのトランスポート層に導入予定機能は既にLinux kernelに(多くがGoogleによって)実装されているものがほとんど。

- RFC 6298 (RTO computation): Linux kernel 3.2 (2012/4/12)
- FACK Loss Recovery/RFC 3782(New Reno Fast Recovery): Linux kernel =< 2.6.12-rc2
- TLP(Tail Loss Probe) (*): Linux kernel 3.10 (2013/3/11)
- RFC 5827 (Early Retransmit) with Delay Timer: Linux kernel 3.5 (2012/5/2)
- RFC 5827 (F-RTO):  Linux kernel 3.5 (2012/5/2)
- RFC 6937 (Propotional Rate Reduction)
- TCP Cubic(*) with options RFC 5681 Linux 2.6.13
- Hybrid Slow Start: 実装 linux kernel(2008/10/29), default有効 linux kernel(2013/10/31)
- RACK(*): linux kernel 4.4(2015/10/16)

(*) は Internet-Draft

QUICは、独自にトランスポート機能を開発するわけではなく、tcpm wgの成果をQUIC仕様に取り込むのが大前提。ただしQUICは向けにアレンジしないといけない部分が出てくるだろう。

どんな部分を変えないといけないのであろうか？

### 4. 外部の方からの質問事項

#### 4.1 Happy Eyeball
RFC 6555(Happy Eyeballs)では、IPv4/IPv6 dual-stack 環境下でのフォールバックが
TCPベースで語られていて、UDPに関しては実現可能性にしか触れられていません。

UDPベースである QUICにはHappy Eyeballs 相当の機能が実装されることが議論されて
いるのでしょうか。それともUDPでの fallbackはアプリ依存の話になり、QUICの仕様
では語られない部分となるのでしょうか。

Ref: https://github.com/shigeki/ask_nishida_about_quic_jp/issues/1

#### 4.2 MTU
QUICってTCPオプションのMSSのような仕組みはありますでしょうか。
IPv4ではフラグメントされてサーバに届くことも多いのでは無いかと危惧しています。
フラグメントされたパケットはフィルタされることもあるかと思います。

DNSにおけるEDNS0の運用上の推奨サイズのようなもので回避する等はあるかと思いますが、
仕様においてQUICのパケットがフラグメントされないようなことを目的としたものはございますでしょうか？

Ref: https://github.com/shigeki/ask_nishida_about_quic_jp/issues/2

#### 4.3 Distinction between QUIC and DDoS
現在，UDP によるDDoS が目立っています．
常時UDP を落とす事業者は少ないですが，有事の際にルーターのようなミドルボックスで

基本UDP はぜんぶ落とす
ただし特定のキャッシュサーバー，権威サーバー宛てのみ53/UDP を許可する
は有効な手段であり，検討している事業者は少なくないと思います．

実装可能性は別として，理論上QUIC とDDoS を仕分けできそうな，トランスポートの特徴などありそうでしょうか？ それとも積極的にTCPフォールバックさせたほうがよさそうでしょうか？

Ref: https://github.com/shigeki/ask_nishida_about_quic_jp/issues/3
