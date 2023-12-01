# PKP-Based_blind_signature for Blockchain in chaumian coin

# 20231201
ブラインド電子署名をやってみたい。

インターネットの匿名性が滅んでいく過程において、それでも匿名で有り続ける領域を残しておきたい。  
匿名性は未来のインターネットにおける可能性の一部だと思っている。  
行列って大学数学では基礎なのに、いろんな応用があって凄いと思う。
きちんとした教科書読むと丁寧な証明が書いてあるけど、本当に軽率ってくらい安易に計算問題として見ているだけで、
それで許されるのが趣味というものだと思っている。

# 20231122
NIST は、PQC を現在の暗号のように、様々なソリューションやシステムで利用したいと考えており、その  
ためには、更なる軽量化が必要であり、2022 年８月に新しいアルゴリズムの公募を開始している。同時に  
NIST は、格子アルゴリズムに基づかない新たな追加の汎用署名スキームを公募している。これは、証明書  
の透明性などの特定のアプリケーションでは、短い署名と迅速な検証を備えた署名スキームが必要なため  
である。  
（以下より引用）  
https://www8.cao.go.jp/cstp/stmain/pdf/20230314thinktank/seikabutsu/shiryou3-1-02.pdf

余談：今マイブームなのは、秘密計算・ブラインド復号というもので秘密鍵の管理の方法に関係している。
安全でないシステムを安全に使うのは難しいし、秘密鍵だけ安全でも意味がないのだがなんとかできないか。

# 20231118（追記）
今日はこのテーマに集中するつもりだったのだが、はじめから挫折。
その後、暗号の安全性理論に興味が移って挫折。

最終的に素体上のパターソン復号にたどり着いた。
誤り位置さえ分かれば決定性アルゴリズムで絵t￥ラーの値を計算できるので、この辺はむやみに多項式をイジる必要がなくなった。
なので今日は寝不足で成果なしである。

素体上ではなく、$F_q:q=3^n$の場合が特に興味深いことがわかる。
ただ公開鍵のサイズが小さくなる利点はあるものの、演算が遅いのが欠点だ。

読めなかった論文が10本近くになった。
これはいかんね。

# 20231118
上にあるCのソースコードは、自分なりに考えたPKPベースの秘密鍵暗号です。
行列部分を符号の生成行列として作りました。
パタリンの効率的なベクトル生成法もあるようなので、それも参考にしたいです。
符号が2次元の正方行列になっているため、符号を超えて行列としても釣っています。
あるいみヒル暗号に近いものがありますが、ヒル暗号のように行列を掛けるだけではなく、
行列の核となるベクトルを選んで、そこに平文を足して、更に符号の生成行列を正則行列として再変換しています。
元の行列が符号語なので、エラーを入れることができるかも知れないですが、復号できるかどうかはまだ確かめていません。
符号語が乱数を代わりを果たしているようにして、そこに平文を足すことで検査行列で符号語を消しつつ、
平文を復元することができました。

まあ何だか色々工夫したはずなんですが、大したことないですね。

本題はこれではなく電子署名なので、まず何に使う電子署名7日を明かしていく作業が必要である。
Fiat-Shamirの動作原理は抱いたわかったので、次はブロックチェーンにブラインド署名を使うことには
どういう意味があるのだろうと考えてみたり。

# 20231117
とりあえずこの問題に取り組むことに決めたのはいいんだが、この先余り関心持たないかも。
でも電子署名の安全性を覚えるのにはいい教材のような気がする。  
難しい数学は一切なしで、電子署名が作れちゃう！（線形代数は必要です）

そういうときに限ってほとんど関係のないweak popov formなんかの論文が読めてしまったりして戻りたくても引き返せない。
電子署名だけを単独でやろうとするのは動機として弱い気がするので、ここはブロックチェーンを新たに開発する目的で取り組みたい。
チャウミアンコインのような。

本来ここには日記的なことは書かないらしいのだが、私の心の掃き溜めはどこにあるのかしら。
頭のいい人には必ず信者がいるので、私はやはり愚かなのだと確信しつつ暗号技術に磨きをかけたい。

昔の研究テーマの暗号の論文を検索すると、なぜか＊SAの委託研究なんか出てきて、オリジナルのIEEEの論文が出てこない。

最近のネット検索は、人が何を調べているのか情報を集めた上で情報を後出ししてくるみたいで気分が悪い。
欲しい情報が何も得られてないのと同じだからだ。
こういう人為的な情報操作がネットでは可能になってしまう。
つまり、その人が見たいものではなく見せたいものを見せる、あるいは見せたくないものも隠すことができるような不自由なネットだ。
それだけでなく端末上の情報も筒抜けのようになっていて、本当に見せる必要のない自分だけの情報はネットに繋げない端末に保存しないと危ないくらいだ。
いわばテクノロジーの放棄である。
IoTっていらなくね？

結局、人為的な操作が困難な健全な情報源といえば、図書館しかなくなるのだろう。
