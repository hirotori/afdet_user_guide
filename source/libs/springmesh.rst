==================
バネ格子
==================

概要
==================
空間格子を移動変形する際に, 極度に歪んだ格子が生じることがある.
形状の歪な格子はしばしば計算の不安定の原因になり,場合によっては発散してしまう場合もあるため,
移動格子による計算手法では格子の質に注意が必要である.

バネ格子はセルの歪な形状を緩和させ,安定して計算を継続させるために用いられる.
本ソルバにおいても基本的なバネ格子法が利用可能であるが,具体的な計算方法を記したものが少ない.
適切な利用と今後のメンテナンスのために,ここにばね格子法の基礎式やその実装について書き記す.

強制変位(1自由度)
=============================
バネ格子の説明に入る前に, バネ格子を利用しない格子の変形方法について説明する.
問題が例えばピストン運動のような1自由度の移動(直線上の運動)の場合は,バネ格子をわざわざ利用せずとも,
以下に述べる方法を用いれば十分に計算可能である.

まず，ある節点(セルを構成するある格子点) :math:`a` の時刻 :math:`t` における座標を :math:`x_{a}(t)` ,
初期座標を :math:`x_{a,0}=x_{a}(0)` と書くことにする．
この格子移動法では，節点の初期座標からの移動量を，**初期座標に依存して決定する**.
:math:`x_{a}(t)-x_{a,0}` を以下のように定める．

.. math:: 
    :label: eq_mesh_1d_move

    x_{a}(t)-x_{a,0} = f(x_{a,0}, t)

ここで :math:`f(x_{a,0}, t)` は任意の関数であるが，多少の制約は存在する．それについては後ほど述べる．
たいていは線形関数

.. math:: 

    f(x_{a,0}, t) = A(t)x_{a,0} + B(t)

が選ばれる. 

例として, 以下のようなピストン運動を考える. 

    左端が :math:`x_{0}=\alpha` , 右端が :math:`x_{0}=\beta\;(\beta > \alpha)` にあるような円管を考える．
    左端を変位 :math:`l(t)` で動かし，右端を固定するとき :math:`\alpha < x < \beta` に存在する節点を動かすための
    移動式の係数 :math:`a(t), b(t)` を求める.


:math:`f(x_{a,0},t) = A(t)x_{a,0} + B(t)` とおく. 左端の節点 :math:`a` の時刻 :math:`t` における **変位** は

.. math:: 

    x_{a}(t)-x_{a,0} = l(t)

に等しい. (絶対位置ではなく変位であることに注意.) よって

.. math:: 

    A(t)\alpha + B(t) = l(t)

が成り立つ. 同様に右端についても

.. math:: 

    A(t)\beta + B(t) = 0

が成り立つので, 連立して :math:`A,B` を求めると,

.. math:: 

    x(t) -x_{0} = \frac{l(t)}{\alpha - \beta}x_{0} - \frac{l(t)}{\alpha - \beta} \beta

となる. (下添え字 :math:`a` は省略.)

関数の制約
---------------------------
上記で述べた方法は節点だけに着目しているため，格子の形状については全く考慮していない．そのため，
場合によってはある節点が隣り合う節点を追い越してしまい，結果として計算の破綻を引き起こすことがある．
そこで，ここでは節点の位置関係を保つためにはどのような条件が必要になるのかについて考察する．
簡単のため，節点が一直線上に存在することを仮定している．

ある2つの点 :math:`x_{a},x_{b}` を考える. ただし :math:`x_{a}(0) < x_{b}(0)` とする. 
節点の位置関係を保つためには,
任意の時刻 :math:`t` においてつねに :math:`x_{a}(t) < x_{b}(t)` を満たさなければならない.
式 :eq:`eq_mesh_1d_move` を用いて,両者の差をとると

.. math:: 

        x_{b}(t) - x_{a}(t) = (x_{b,0} - x_{a,0}) \left( 1 + \frac{f(x_{b,0},t) - f(x_{a,0},t)}{x_{b,0} - x_{a,0}}\right)

となるので, 要求を満たすためには

.. math:: 
    :label: eq_mesh_move_constraint

    \frac{f(x_{b,0},t) - f(x_{a,0},t)}{x_{b,0} - x_{a,0}} > -1

でなければならない. 

もし2つの点として :math:`x_{0}, x_{0} + \delta x_{0}` をとると, :math:`f(x_{0} + \delta x_{0},t)=f(x_{0},t)+\delta f` とおいて, 
:eq:`eq_mesh_move_constraint` より

.. math:: 

    \frac{\delta f}{\delta x_{0}} > -1

が成り立つ. :math:`\delta x_{0} \rightarrow 0` の極限をとると,

.. math:: 

    \frac{df}{dx} > -1

となる. 1自由度の運動を 式 :eq:`eq_mesh_1d_move` で与える場合は,上の条件に留意する必要がある.

1自由度の運動であれば, 上記の強制変位法で大抵は解決できる．しかし，運動が2次元や3次元的になると，この方法はうまくいかない可能性が高い．
より複雑な運動に対応するために，バネ格子法を用いる．


バネ格子法
==================
バネ格子は大まかに2つの方法, vertex spring と segment spring, に分けられる [1]_.
前者はバネの長さが0で無い限り常に張力を生じる（バネの長さが0のときつりあう）モデルで,
主にメッシュのスムージングに用いられる. 後者は対照的に，変形前の初期状態で釣り合うと仮定し，
節点が移動することにより張力を生じるモデルである．移動格子の方法としてのバネ格子は主に後者が用いられるので,
ここでは後者について説明する.

vertex spring も segment springモデルも, 要素の辺をバネと見立てることは共通である. 両者の違いは釣合いの考え方である.
前者はバネの平衡長さ(equilbrium length)をゼロとする. 一方で後者は平衡長さは初期(移動前)の辺長である.

vertex spring method
----------------------
ある時刻の :math:`i` 番目の節点座標を :math:`\boldsymbol{x}_{i}` とおく.
節点 :math:`i` に隣接する節点を :math:`j` とおくと, 辺 :math:`ij` 間に働くバネによる復元力は

.. math:: 

    \boldsymbol{f}_{ij} = \alpha_{ij} (\boldsymbol{x}_{j} - \boldsymbol{x}_{i})

と表される. :math:`\boldsymbol{x}_{i}=\boldsymbol{x}_{j}` のとき復元力はゼロである.
隣接する節点は普通は複数個有るので, 合力は

.. math:: 
    :label: eq_vertex_spring

    \boldsymbol{F}_{ij} = \sum_{j \sim nb(i)} \boldsymbol{f}_{ij} = \sum_{j \sim nb(i)}\alpha_{ij} (\boldsymbol{x}_{j} - \boldsymbol{x}_{i})

となる. 新しい節点座標は, この式を基に, 以下の反復法を用いて求められる [1]_.

.. math:: 
    :label: eq_vertex_spring_iter

    \boldsymbol{x}_{i}^{k+1} = \frac{\sum_{j \sim nb(i)}\alpha_{ij}\boldsymbol{x}_{j}^{k}}{\sum_{j \sim nb(i)}\alpha_{ij}}


なお :math:`k` は反復インデックスである. 普通この方程式を厳密に解くことは無い.  
節点には境界面上の節点も含まれるが, 反復の間は固定される.

式 :eq:`eq_vertex_spring_iter` の物理的な解釈を述べる. 式 :eq:`eq_vertex_spring` を基に釣合方程式を立てると,

.. math:: 

    -\sum_{j \sim nb(i)}\alpha_{ij} \boldsymbol{x}_{i} + \sum_{j \sim nb_{inner}(i)}\alpha_{ij}\boldsymbol{x}_{j} = -\sum_{j \sim nb_{bnd.}(i)}\alpha_{ij}\boldsymbol{x}_{j}

とかける. 左辺は領域内部の, 右辺は境界面上の節点についての和である. 全ての内部の節点 :math:`i` についてこれが成り立たねばならない.
これは連立方程式

.. math:: 

    \boldsymbol{A}\boldsymbol{X}=\boldsymbol{B}

を解くことと同じである. これを直接解くのは計算コストが著しいので, Jacobi法により解を求める. その式こそ :eq:`eq_vertex_spring_iter` である.

vertex springは新しい座標を周囲の座標の重み付き平均で求めることと同じである. 重み係数 :math:`\alpha_{ij}` は普通

.. math:: \alpha_{ij} = 1 \quad \forall i,j 

とする [1]_ . したがって, 新しい節点座標は周囲の節点座標の算術平均である. これは Laplacian smoothing と同じである.


segment spring method
------------------------
segment springでは, 節点の座標では無く節点の変位に対してフックの法則が適用される.
移動前の節点 :math:`i,j` の位置座標を :math:`\boldsymbol{x}_{i}^{n},\boldsymbol{x}_{j}^{n}` とし,
変位を :math:`q_{i},q_{j}` , 移動後の位置座標を :math:`\boldsymbol{x}_{i}^{n+1}=\boldsymbol{x}_{i}^{n}+\boldsymbol{q}_{i},\boldsymbol{x}_{j}^{n+1}=\boldsymbol{x}_{j}^{n}+\boldsymbol{q}_{j}` とおく.

節点 :math:`i` に働く 節点 :math:`j` からの力 :math:`\boldsymbol{F}_{ij}` は, 次式で表される [1]_.

.. math:: 
    :label: eq_segment_spring

    \boldsymbol{F}_{ij} &= k_{ij} \left\{
    (\boldsymbol{x}_{j}^{n+1}-\boldsymbol{x}_{i}^{n+1}) - (\boldsymbol{x}_{j}^{n}-\boldsymbol{x}_{i}^{n})
    \right\} \\
    &= k_{ij} (\boldsymbol{q}_{i} - \boldsymbol{q}_{i})

これもvertex spring と同様に, 反復的に上式を解いて変位を得る.

.. math:: 

    \boldsymbol{q}_{i}^{k+1} = \frac{\sum_{j \sim nb(i)}k_{ij}\boldsymbol{q}_{j}^{k}}{\sum_{j \sim nb(i)}k_{ij}}


ここで, 境界面上の節点に対してはDirichlet条件として扱い, あらかじめ変位量を与えておく.
ばね定数は, 通常は節点間の長さの逆数が与えられる. 

.. math:: 

    k_{ij} = \frac{1}{\left\|\boldsymbol{x}_{j}^{n} - \boldsymbol{x}_{i}^{n}\right\|}


長さの逆数をとることで, 格子点が密な領域ではバネを硬くして大変形を妨げ, 格子の破綻を避けることが出来る.
この選択が合理的であることは, 1次元問題の場合にBlom [1]_ により示されている.

反復の後で, 新しい節点座標が以下の式により計算される.

.. math:: 

    \boldsymbol{x}_{i}^{n+1} = \boldsymbol{x}_{i}^{n} + \boldsymbol{q}_{i}^{k_{end}}


上式では, :math:`x` 方向の変位は :math:`x` 方向の力にのみ作用する. :math:`y` も同様である. 
しかし厳密には, トラス構造の例でわかるように, :math:`x` および :math:`y` 方向の力は :math:`x,y` 方向の変位両方に依存する. 
したがって上式は簡略化されたバネの釣合いを考えていることになる. 
厳密なバネの釣合いを考える例は例えば Farhat [2]_ らなどがある.

ねじりバネの方法
=================
非構造三角形格子の変形で大きな問題となるのは, セルの反転である. 
三角形格子の場合, 三角形の頂点が反対側の面を通過し, セルが反転してしまうという難点がある. (これはsnap-throughと呼ばれている.)
Farhat [2]_ らはこれを解決するため, 三角形の辺の端(つまり頂点上)にねじりバネを設置し, 辺のねじれを考慮した.
Farhatらのねじりバネ定数は以下のように定義される.

.. math:: 
    :label: eq_torsional_spring

    k_{i} = \frac{1}{\sin^{2}\theta_{i}} 


ここで :math:`\theta_{i}` は :numref:`fig_triangle` が示すように, 節点 :math:`i` 周りの角度である. 
角度がゼロに近づくほどバネの剛性は上がり, 格子が潰れにくくなる.

.. figure:: ../images/diagram-1.png
    :name: fig_triangle

    節点, 辺, 角度の定義

なお, 角度 :math:`\theta_{i}` は三角形の面積の公式から求められる. すなわち, 面積を :math:`A_{ijk}` とおくと

.. math:: 

    A_{ijk} = \frac{1}{2}l_{ij}l_{ki}\sin\theta_{i}

で求められるので, したがってねじりばね定数は

.. math:: 

    k_{i} = \frac{1}{\sin^{2}\theta_{i}} = \frac{l_{ij}^{2}l_{ki}^{2}}{4A_{ijk}^{2}}

より求められる. 

Farhatらの論文では, このねじりバネによるねじりモーメントを力に線形変換し, 通常の線形バネに重ね合わせて
使用している. 

半ねじりバネの方法
=====================================
Farhatらの方法は3次元問題における四面体要素にも拡張されている [3]_ が, 定式化が煩雑となるため計算コストが増加する. 
これに対し, 村山ら [4]_ は三辺それぞれの線形バネについて,その対角を 式 :eq:`eq_torsional_spring` で表されるバネ定数として加える定式化を提案した.
すなわち, 2次元三角形格子においては, 辺 :math:`l_{ij}` におけるばね定数を

.. math:: 

    k_{ij} = \frac{1}{\left\|\boldsymbol{x}_{j}^{n} - \boldsymbol{x}_{i}^{n}\right\|} + \sum_{k} \frac{1}{\sin^{2}\theta_{k}}


とする. 和記号は, 辺を共有する全ての三角形の対角についての和である.
3次元問題における四面体要素については, :numref:`fig_tetra_spring` のように, ねじれの位置にある2つの面がなす角 :math:`\theta_{kli-klj}`  に関するばね定数を加える.

.. figure:: ../images/tetra_spring.png
    :name: fig_tetra_spring
    :align: center

    四面体におけるバネ


ソルバでの取り扱い
====================
本ソルバでは, 村山らの半ねじりバネの方法を援用した segment spring model を採用している. 
ただし村山等の報告と異なり, Farhatらの2次元問題のねじりばね定数(すなわち,2辺角)も加えている. したがって, 辺 :math:`l_{ij}` に対するばね定数は

.. math:: 

    k_{ij} = \frac{1}{\left\|\boldsymbol{x}_{j}^{n} - \boldsymbol{x}_{i}^{n}\right\|} + \sum_{k} \frac{1}{\sin^{2}\theta_{k}^{edge}} + \sum_{k} \frac{1}{\sin^{2}\theta_{k}^{plane}}

となる. また, 反復は変位ベクトルでは無く位置ベクトルに対して行う. 反復 :math:`k` 回目の更新で得られた座標を
:math:`\boldsymbol{x}_{i}^{k} = \boldsymbol{x}_{i}^{n} + \boldsymbol{q}_{i}^{k}` とおき, 式 :eq:`eq_segment_spring` に代入すると

.. math:: 

    \boldsymbol{x}_{i}^{k+1} = \boldsymbol{x}_{i}^{n} + \frac{\sum_{j \sim nb(i)}k_{ij}\left(\boldsymbol{x}_{i}^{k} - \boldsymbol{x}_{i}^{n}\right)}{\sum_{j \sim nb(i)}k_{ij}}

と表される.

クラス
---------------------
バネ格子に関する処理は ``SpringMesh_t`` クラス(``SpringMesh.f90``)が受け持つ. 


References
===============
.. [1] F.J. Blom, Considerations on the spring analogy, International Journal for Numerical Methods in Fluids. 32 (2000) 647–668. 

.. [2] C. Farhat, C. Degand, B. Koobus, M. Lesoinne, Torsional springs for two-dimensional dynamic unstructured fluid meshes, Computer Methods in Applied Mechanics and Engineering. 163 (1998) 231–245. 

.. [3] C. Degand, C. Farhat, A three-dimensional torsional spring analogy method for unstructured dynamic meshes, Computers & Structures. 80 (2002) 305–316. 

.. [4] 村山光宏, 非構造動的移動格子に関する研究, 航空宇宙技術研究所特別資料: 航空宇宙数値シミュレーション技術シンポジウム2001論文集 = Special Publication of National Aerospace Laboratory: Proceedings of Aerospace Numerical Simulation Symposium 2001. 53 (2002) 161–166.
