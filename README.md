# PKP-Based_blind_signature for Blockchain in chaumian coin

cf. PKP-Based Signature Scheme : Beullens, Faugere, Koussa, Macario-Rat, Patarin, Perret

# 20240218
オリジナルバージョン  
ベクトルPKP（VPKP-IDS）:  
１．公開鍵の生成  
1-1.置換群$$\pi$$を生成する。また、秘密のベクトルXを生成して秘密鍵とする。  
1-2.自己同型群の位数が32以上であることを確認する。  
1-3.そうでなければ1に戻る。  
1-4.ベクトルに置換を繰り返しかけて、ラウンドごとに差分の和
$$\Sigma_{n=0}^{32}(X_i \verb|^| X_{(i+1)%N}$$をとったものを$$\alpha$$とする。  
$(\pi,\alpha)$を公開鍵、Xを秘密鍵とする。

２．認証  
2-1.32バイトの乱数ベクトルを生成し、それをRとする。  
2-2.Rに置換をかけ、その差分の和$$\beta$$を検証者に送信する。  
2-3.検証者は０か１を証明者に送る。  
2-4.検証者のチャレンジが０の時、  
証明者はRを送る。  
検証者のチャレンジが１の場合、証明者は$$X \verb|^| R$$  を送信する。  
検証者は$$X \verb|^| R$$に公開置換をかけて$$\alpha \verb|^| \beta$$であることを確認する。


# 20240214
PKP-DSSの署名サイズは70キロバイト超えるよという話。

# 20240130
PKP-IDSからPKP-DSSの流れ。  

**実装はこの通りじゃありません。独自解釈が入っています。オリジナルはもう少し後で書きます。**

## PKP-IDS：  
秘密鍵をsk、公開鍵をpkとする。この時、  

$sk=\pi,pk=(v,\alpha=sum(v[\pi(i)]))$  

整数ベクトルの隣接する要素の差の総和をとることでハッシュの代わりとしている。  
２つの置換$$P,Pa$$を$$P_k=P^{k}[i]$$とし、$$P_k$$をベクトル$v$にかけて$$v[i]^=v[P_k[i]]$$とし、$v$をラウンドごとに更新し、
このベクトル$v$の要素の差分の総和$$\sigma(v[P[i]^v[P[(i+1)%N])$$を署名ベクトルとする。  
ここでベクトルは小さな素数pを、例えば$p=4093$位にとり、置換の次元を$N=106$位にとる。
rはランダムベクトルとする。

1.  
$c0=sum(r[i]+r[(i+1)%N])$  
$c1=sum(r[\phi(i)]\textasciicircum r[\phi(i+1)%N])$  

return c0,c1

検証：  
2.  
 c <- F_p  
 
3.  
$z[i]=r[\phi(i)]+cv[\pi(\phi(i))]$  
return z

4.  
if b <- 0  
$return \phi$  
if b <- 1  
$return \pi\phi$

5.  
if b=0  
$c_0==sum(z[\phi(i)^{-1}])+\alpha$  
if b=c  
$c_1==sum(z[i]-cv[\pi\phi(i)])$  

PKP-IDSの流れは、こんな感じである。  

## PKP-DSS:  
署名作成：  

\phi(1..289)：ランダム置換  
r(1..289)：ランダムベクトル  
$c_0(1..289)$  
$c_1(1..289)$  

$com = H((c_0,c_1)(1..289))$

c1,c2,...,c289 <- H(m||com)：ハッシュ関数からｃを作成  
rsp1=z(1..289)  

b1,b2,..,b289 <- H(m||com||z)：ハッシュ関数からチャレンジビットを生成

$rsp2=((b*\pi)\phi(1..289) || c_{1-b}(1,..,289))$

return (com,rsp1,rsp2)  

署名検証：（m,(com,rsp1,rsp2)）  

c(i) <- H(m||com)  
b(i) <- H(m||com||rsp1)

rsp1 -> z(i)  
$rsp2 -> (b*\pi\phi(i), c_{1-b}(i))$

... orz（理解不足）

PKP-DSSのサイズを小さくできないかと思って作った爆笑署名。この問題の背景には、置換群の持つ軌道が、その置換に対して固有であり１対１対応になるだろうという予測に基づいている。  
多分これから証明する上で欠陥が分かってくると思う。というか証明なんか理解できないかもしれないのに。  
で、例えば次元$$N$$の巡回置換を例にとろう。Pが位数ｎの置換であるとき、その置換の軌道をラウンドごとの置換の連接にハッシュをとったものに対応付ければいい。  
すると、同じ位数でも置換の位置が変われば、異なるハッシュ値が求められるので、任意の置換の軌道はその置換に固有なものであるということが分かるはずだ。

# 20240129
ファイルの説明:

pace.c:PKP-DSSの奇標数の拡大体バージョン。わざと動かなくしている。  
peace.c:PKPで雑音を消して平文を復号するようにしたつもりの暗号。秘密鍵なのに平文が膨らむのを何とかしたい。単にカーネルに含まれるベクトルを平文にかぶせるだけでいいのかも？  

sig.c:  
秘密のベクトルを$w$とし、更に秘密の置換$Pa$をの逆置換$\inv_Pa$とし、$v[i]=w[inv_Pa[i],x=\sigma(v[P[i]]^v[P[(i+1)%N]])$とし、$v$を公開ベクトル、更に公開置換を$P$とする。
ここで公開鍵は$(P,v,x)$である。

多分この電子署名を最後に暗号からは手を引くと思う。  
最後に、PKP-DSSのブラインド署名バージョンを作って終わりたい。

今日も適当に答え合わせして疲れたので動くところでアップロードした。  
というか、証明はおろか数式の使い方すらわからない自分が何を言っても伝わらないだろうし、数学のような概念を扱う学問は自分には合わないのだと分かった。  
公開しているのは、公開しないとログインできない時にクローン出来なくなるからだ。

出来そうにもないことを趣味にしてしまったのが運の尽きだと思う。  
理解するというより、目隠ししながら走り回っているみたいと言われる。

これからはちょっと論理学をやって、数学基礎論（多分無理）を覗いて、Haskellなんかをいじって、自動証明みたいなことをやろうとしたり。  
何で論理学かというと、ゼロ知識証明でいうところの健全性とか完全性というのが論理学の用語だと知ったからだ。  
これも全て数学の証明の正しさの根拠を理解するためである。

ダメなら歴史の研究とかをやってみたい。
マクニール万歳！ｗ

# 20231215
久しぶりにプログラム。
群論の語の問題を計算するために始めた。
しかし誤の問題を正しく理解していないと計算さえできないので勉強中。
まるで当て外れなことをしているのではないかと気になったけど、外れでもないことが分かり何とかなりそう。
変な方向に行くとしたらちゃんとした知識がないからで、英語の本になるけどネットに落ちていた紙の本だと1万円以上するpdfを読めばいい。

語の問題から新しいテーマは始まる。

# 20231204
組み合わせの生成にcharが使われていて、256以上の次元の置換が生成できなかったところをshortに修正。  
いつになったら最新性能のハードの凄さを体感できるのだろうと思っているが、今の流れからすると永遠にその時は来ない気がする。

# 20231201
ブラインド電子署名をやってみたい。

インターネットの匿名性が滅んでいく過程において、仮にその結果が絶滅でも匿名で有り続ける領域を残しておきたい。  
匿名性は未来のインターネットにおける可能性の一部だと思っている。  

行列って大学数学では基礎なのに、いろんな応用があって凄いと思う。
PKPのランダム公開行列を作れるようになった。

趣味っていうだけあって公開しないということもできるんだけど、パソコンが壊れて予備のメディアもないときに
ネットにバックアップがあるのって便利だと思うし、ログインできないときでもクローンできるように公開してるだけって感じ。
なるべくGAFAには頼りたくないし、MSもそのうちだなんだけど、だったらGitLabにもクローン置いとけみたいな。

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
