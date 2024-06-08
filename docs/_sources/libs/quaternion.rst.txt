=================
クォータ二オン
=================

クォータ二オンを直接触ることは無いだろうが, ``Rigidbody_t`` を
扱う上で必要なのでここで整理する. 

定義
==================
クォータ二オンは複素数を拡張した数の体系で, 虚数単位 :math:`i,j,k` を持ちいて

.. math:: 

    \tilde{q} = a + bi + cj + dk \quad (a,b,c,d \in \mathbb{R})

と表される数である. :math:`i,j,k` はクォータ二オンが定義される空間の基底であり, 

.. math:: 

    i^{2} = j^{2} = k^{2} = ijk = -1

を満たす. また, これを用いれば相異なる基底間の積を定義出来る. 

.. math:: 
    ij &= -ji = k \\
    jk &= -kj = i \\
    ki &= -ik = j \\

クォータ二オンの :math:`a` をスカラー部, :math:`bi + cj + dk` をベクトル部という. :math:`i,j,k` を
改めて :math:`\mathbb{R}^{3}` の基底だと思うと, ベクトル部は実ベクトル空間の元 :math:`\boldsymbol{v}` である. この観点から
クォータ二オンを

.. math:: 

    \tilde{q} = a + \boldsymbol{v}

と書くことにする. 例えば実ベクトル空間のベクトルは :math:`a=0` のクォータ二オンである. 

基本的な演算
=================

相異なる2つのクォータ二オン 

.. math:: 

    \tilde{q} &= q_{0} + q_{1}i + q_{2}j + q_{3}k = q_{0} + \boldsymbol{q} \\
    \tilde{p} &= p_{0} + p_{1}i + p_{2}j + p_{3}k = p_{0} + \boldsymbol{p}  \\

の演算について述べる. 

和
------------

和については成分毎の演算である. 

.. math:: 

    \tilde{q} + \tilde{p} = q_{0} + p_{0} + \boldsymbol{q} + \boldsymbol{p}

ハミルトン積
-------------

積については分配律から計算する. 分配法則によって展開すると

.. math:: 

    \tilde{q}\tilde{p} = q_{0}p_{0} + q_{0}\boldsymbol{p} + p_{0}\boldsymbol{q} + \boldsymbol{q}\boldsymbol{p}

ここで :math:`\boldsymbol{q}\boldsymbol{p}` は基底間の積のルールにしたがって計算する. 
計算過程は省略するが, 実ベクトル空間に対する演算子である内積と外積を用いて次のように書ける. 

.. math:: 

    \boldsymbol{q}\boldsymbol{p} = -(\boldsymbol{q}\cdot\boldsymbol{p}) + \boldsymbol{q}\times\boldsymbol{p}

クォータニオンの積は非可換であることに注意する. 

共役
------------

:math:`\tilde{q}` の共役とはベクトル部を異符号にしたものであり

.. math:: 

    \tilde{q}^{\ast} = q_{0} - \boldsymbol{q}

で表される. 2つのクォータニオンの積の共役は次式となる. 

.. math:: 

    (\tilde{q}\tilde{p})^{\ast} = \tilde{p}^{\ast}\tilde{q}^{\ast}


ノルム
-----------------

クォータニオンのノルムは共役クォータニオンを用いて定義することも出来る. 

.. math:: 

    \left\| \tilde{q} \right\| = \sqrt{q_{0}^{2}+q_{1}^{2}+q_{2}^{2}+q_{3}^{2}}
     = \sqrt{\tilde{q}\tilde{q}^{\ast}} = \sqrt{\tilde{q}^{\ast}\tilde{q}}

逆クォータニオン
------------------

共役を用いて逆クォータニオンを定義出来る. 

.. math:: 

    \tilde{q}^{-1} = \frac{\tilde{q}^{\ast}}{\left\| \tilde{q} \right\|}

実際, :math:`\tilde{q}\tilde{q}^{-1} = \tilde{q}^{-1}\tilde{q} = 1` であることがわかる. 


ベクトルの回転
===================

回転角 :math:`\theta` とし, ノルムが1となるようなものとして

.. math:: 

    \tilde{q} = \cos \frac{\theta}{2} + \boldsymbol{n}\sin \frac{\theta}{2}

を定義し, これを回転クオータニオンと呼ぶ. ただし :math:`\boldsymbol{n}` は回転軸を向く単位ベクトルである. 

このような :math:`\tilde{q}` を用いると, 実ベクトル :math:`\boldsymbol{r}` の回転後のベクトル :math:`\boldsymbol{r}'` は次式で表される. 

.. math:: 

    \boldsymbol{r}' = \tilde{q}\boldsymbol{r}\tilde{q}^{\ast}

まず一般の :math:`\tilde{q}` について計算すると次式となる. 

.. math:: 

    \tilde{q}\boldsymbol{r}\tilde{q}^{\ast}
    =
    [q_{0}^{2}-(\boldsymbol{q}\cdot\boldsymbol{q})]\boldsymbol{r}
    + 2q_{0}(\boldsymbol{q}\times\boldsymbol{r})
    + 2(\boldsymbol{q}\cdot\boldsymbol{r})\boldsymbol{q}

したがって回転クオータニオンの場合は

.. math:: 

    \tilde{q}\boldsymbol{r}\tilde{q}^{\ast}
    =
    (\boldsymbol{n}\cdot\boldsymbol{r})\boldsymbol{n}
    + [\boldsymbol{r} - (\boldsymbol{n}\cdot\boldsymbol{r})\boldsymbol{n}]\cos\theta
    + \boldsymbol{n}\times\boldsymbol{r}\sin\theta

で表される. 幾何学的なイメージは2番目の文献を参照のこと. 

回転の行列表現
------------------

ベクトルの回転を行列で表す. すなわち, 

.. math:: 

    \boldsymbol{r}'
    = \tilde{q}\boldsymbol{r}\tilde{q}^{\ast}
    = {\bf A} \boldsymbol{r}

を満たすような :math:`{\bf A}` を求める. 計算過程は省略するが, 次のようになる. 

.. math:: 

    {\bf A} 
    = 
    \begin{bmatrix}
    q_{0}^{2}+q_{1}^{2}-q_{2}^{2}-q_{3}^{2} & 2(q_{1}q_2-q_0q_3) & 2(q_2q_0+q_1q_3) \\
    2(q_{1}q_2+q_0q_3) & q_{0}^{2}-q_{1}^{2}+q_{2}^{2}-q_{3}^{2} & 2(q_2q_3-q_1q_0) \\
    2(q_1q_3-q_2q_0) & 2(q_2q_3+q_1q_0) & q_{0}^{2}-q_{1}^{2}-q_{2}^{2}+q_{3}^{2}
    \end{bmatrix}

:math:`\tilde{q}` が回転クォータニオンの場合, 対角成分は次のようになる. 

.. math:: 

    A_{11} &= 2q_0^2 + 2q_1^2 - 1 = 1 - 2q_2^2 - 2q_3^2 \\
    A_{22} &= 2q_0^2 + 2q_2^2 - 1 = 1 - 2q_3^2 - 2q_1^2 \\
    A_{33} &= 2q_0^2 + 2q_3^2 - 1 = 1 - 2q_1^2 - 2q_2^2 \\

例えば :math:`A_{11}` については, 

.. math:: 

    A_{11} &= q_{0}^{2}+q_{1}^{2}-q_{2}^{2}-q_{3}^{2} \\
    &= \cos^2\frac{\theta}{2} + \sin^2\frac{\theta}{2}(n_x^2-n_y^2-n_z^2) \\
    &= \cos^2\frac{\theta}{2} - \sin^2\frac{\theta}{2}(n_x^2+n_y^2+n_z^2) + 2\sin^2\frac{\theta}{2}n_x^2 \\
    &= 2\cos^2\frac{\theta}{2}+ 2\sin^2\frac{\theta}{2}n_x^2 -1 \\
    &= 2q_0^2 + 2q_1^2 - 1 \\
    &= 2q_0^2 + 2(1-q_0^2-q_2^2-q_3^2) - 1 \\
    &= 1 - 2q_2^2 - q_3^2

として示される. 他の成分も同様である. 

回転の合成
------------------

初期座標を :math:`\boldsymbol{r}_{0}` とおく. 1回目の回転を, 回転クォータニオン
:math:`\tilde{q_{1}}` で, 2回目を :math:`\tilde{q_{2}}` で表すと, 2回回転したあとのベクトル :math:`\boldsymbol{r}_{2}` は

.. math:: 

    \boldsymbol{r}_1 &= \tilde{q_{1}}\boldsymbol{r}_0\tilde{q_{1}}^{\ast} \\
    \boldsymbol{r}_2 &= \tilde{q_{2}}\boldsymbol{r}_1\tilde{q_{2}}^{\ast} 
     = (\tilde{q_{2}}\tilde{q_{1}})\boldsymbol{r}_0(\tilde{q_{2}}\tilde{q_{1}})^{\ast}

で表される. 一般にn回転後のベクトル :math:`\boldsymbol{r}_{n}` は, 

.. math:: 

    \boldsymbol{r}_n = \tilde{Q}_n\boldsymbol{r}_0\tilde{Q}_n^{\ast}

で表される. ただし :math:`\tilde{Q}_n = \tilde{q}_{n}\tilde{q}_{n-1}\cdots \tilde{q}_{1}` 

プログラム上では, 回転の度に全てのクォータニオンを掛け直すのではなく, 
n-1回転目の回転クォータニオン :math:`\tilde{Q}_{n-1}` を記憶しておいて

.. math:: 

    \tilde{Q}_n = \tilde{q}_{n}\tilde{Q}_{n-1}

のようにして演算すれば良い. 

球面補間(Slerp)
=================

参考
==================
* Wikipedia, 四元数, https://ja.wikipedia.org/wiki/%E5%9B%9B%E5%85%83%E6%95%B0
* 谷田部 学, クォータニオン計算便利ノート, https://www.mss.co.jp/technology/report/pdf/18-07.pdf
* https://qiita.com/aa_debdeb/items/3d02e28fb9ebfa357eaf
* https://risalc.info/src/quaternion-rotation.html