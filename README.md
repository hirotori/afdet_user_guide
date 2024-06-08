# Info
## 開発環境
`Sphinx` のインストール. https://planset-study-sphinx.readthedocs.io/ja/latest/02.html を参考に.

テーマは `read_the_docs`. これも追加でインストールする必要がある. [上記リンクのこのページ](https://planset-study-sphinx.readthedocs.io/ja/latest/06.html) を参考に.

## 記号の統一
  - ベクトルは太字. `\boldsymbol` か `\bf` 使うかまだ統一出来ていない...
  - 頂点座標も $\bm{r}$ , $\bf{r}$ , $\bm{x}$ などが混じっている.


## 思想
### 文書にすることの意味
1. (情報の共有) 文書は何年も残るので確実に情報が伝わる. 間違いがあれば継ぎ足したり修正もできる．

2. (研究遂行の円滑化) コードから実現したい処理を読み解くのは勉強にはなるが, 研究に費やす時間は限られるので，あまり読解ばかりに時間を割くことはできない．読解に必要な情報を提供することで, プログラムの理解は早くなる．結果として，研究に割ける時間は増え，成果も出しやすくなる. 
   
3. (背景知識の理解促進)　APIだけ読んでもどんな意図かはわかりにくい．物理的な意味や，背景にある計算手法について文書にまとめることで，理解することが容易になる．


### どこまで書くべきか
あまり項目を増やしすぎるとメンテナンスが大変になる. また, 研究手段とは言え, コードに関してある程度自力で解決出来るようにもなって欲しい.　もともと個人的メモで書きためていたので, 細かすぎる部分も多い(特に `/source/lib` ディレクトリ以下の内容). こういうドキュメントに往々にして起こる事態だが, プログラムの更新に追いつかなくなると修正が面倒に成りやがて放置される(らしい). 

そこで, おおむね以下のガイドラインに沿って整備を進めたい. なお現状これに完璧に沿っているわけでは無い.

1. インターフェースや依存関係の記述, APIは基本的に `Doxygen` のようなコードリファレンスに任せる. これによって, 本ドキュメントの対象範囲を，コードの根拠となる計算スキームや物理学的な背景などの解説に絞ることが出来る.

2. 個別のテーマに沿った内容は書かない．あくまで，ソルバに備わった機能に着目する．個別のテーマの内容は修論を読めばわかるとみなして，触れなくてもよいと考えている. 明確に参考資料が分かっている場合はそれを読んで各自勉強すればよいし, そうするべき.
逆に言えば, 参照可能な資料がない or 残されていない事柄についてサポートが必要. 例えば，幾何学量の評価について触れている修論はほぼない．非構造格子に特有のデータ構造などの詳細な情報もない．これらの情報を提供することで，より円滑にプログラムを理解し，研究を進める手助けをしたい．

### どのように書くべきか
1. 図は出来るだけ多い方が良い. メッシュのデータ構造や, ベクトルなどは図の方がわかりやすい.

2. できるだけ参考文献を挙げて, それに基づいて書きたい. これは各自が参考文献を開いて勉強するための指針, 道標の役割も本ドキュメントに期待しているから.

3. ソルバを使う人の間で, ある程度用語を共通化したい. お互いに通じる用語があればコミュニケーションもより円滑になる. 新しくメモを残す際の参考になれば.

### どのように共有するべきか?
1. publicなリポジトリにして自由にダウンロード. github pagesを活用. 
2. privateなリポジトリで, ユーザアカウント知ってる人だけダウンロード可能. 手っ取り早いがややメンドクサイ.
3. driveにzipファイルを置いておく. 更新忘れそう.
4. ネットにアップロード. 1.と同じ.
5. 限定して(例えば学内のみ, 研究室LANのみ)公開. やり方がわからないし, 公開用サーバーはどう手配する?