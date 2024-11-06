==================
非圧縮性流体の計算
==================


基礎方程式
=====================
非圧縮性流体の支配方程式は, 以下の連続の式 (質量保存則):

.. math::
    :label: conti_eq

    \nabla\cdot{\boldsymbol u} = 0

と, 運動量保存則:

.. math::
    :label: momentum_eq

    \frac{\partial \boldsymbol{u}}{\partial t}+\nabla\cdot\left(\boldsymbol{uu}\right) = -\frac{1}{\rho}\nabla p+ \nabla\cdot\left(\nu\nabla\boldsymbol{u}\right)

の2つである. 圧力 :math:`p` の時間発展項が無いため， 時間進行に工夫が必要となる．これまでに様々な時間進行法が提案されている．
最も有名なのは MAC法 (とその系列), Fractional step法, そしてSIMPLE法 (とその系列) である. 
これらの解法について網羅的に説明することはここでは避ける. 詳細な解説は文献 [1]_ [2]_ [3]_ [4]_ を参照いただきたい.

Fractional step法
==================
本ソルバはFractional step法を用いて速度と圧力を時間発展させる. 以下, 簡単に概要を述べる. 
連続の式は発散オペレータを用いて

.. math::
    :label: div_eq

    D(\boldsymbol{u}) = 0

と表す. また, 運動量保存則は便宜上, 空間項をまとめて次のように書く. 

.. math::
    \frac{\partial \boldsymbol{u}}{\partial t} = f(\boldsymbol{u},p)

なお, 右辺は

.. math::
    f(\boldsymbol{u},p) = G(p) + D(\boldsymbol{uu}) + L(\boldsymbol{u})


で, :math:`G,D,L=DG` はそれぞれ勾配, 発散, ラプラシアンのオペレータである. 簡単のため, 時間に関して速度は1次前進差分とする. 圧力については陰的差分をとると

.. math::
    \frac{\boldsymbol{u}^{n+1} - \boldsymbol{u}^{n}}{\Delta t} = f(\boldsymbol{u}^{n},p^{n+1})


とかける．ここで :math:`u^{n} = u(t^{n})=u(n\Delta t)` である. Fractional step法では, まず上式を2段に分離する. 

.. math::
    :label: dummy_velocity_eq

    \frac{\boldsymbol{u}^{\ast} - \boldsymbol{u}^{n}}{\Delta t} = f(\boldsymbol{u}^{n},0)

.. math::
    :label: next_velocity

    \frac{\boldsymbol{u}^{n+1} - \boldsymbol{u}^{\ast}}{\Delta t} = f(0,p^{n+1}) = G(p^{n+1})


圧力を省いた :eq:`dummy_velocity_eq` から仮の速度を算出する. そして :eq:`next_velocity` を用いて, 仮速度と圧力から次の段階の速度を求める. 
圧力は :eq:`next_velocity` の速度 :math:`\boldsymbol{u}^{n+1}` が連続の式を満たすように決める. 連続の式 :eq:`div_eq` に代入し整理すると, 

.. math::

    0 = D(\boldsymbol{u}^{\ast}) + \Delta t L(p^{n+1})

を得る. これは圧力 :math:`p^{n+1}` に関するPoisson方程式である. これを解いて :math:`p^{n+1}` を求めたあと, :eq:`next_velocity` より
新しい速度 :math:`\boldsymbol{u}^{n+1}` を得る. 


**References**

MAC法, SMAC法, Fractional step法の解説はこれが詳しい.

.. [1] 梶島岳夫, 乱流の数値シミュレーション 改訂版, 養賢堂, 2017
.. [2] J.H. Ferziger, M. Perić, and R. L. Street. Computational methods for fluid dynamics. springer, 2019.

以下2つはSIMPLE法の解説が詳しい. 

.. [3] F. Moukalled, L. Mangani and M. Darwish. The finite volume method, Springer International Publishing, 2016.
.. [4] H.K. Versteeg, W. Malalaekera. 松下洋介, 斎藤泰洋, 青木秀之, 三浦隆利訳. 数値流体力学 第2版, 森北出版, 2011. 