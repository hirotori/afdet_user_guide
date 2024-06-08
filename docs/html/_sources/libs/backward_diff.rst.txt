====================
後退差分クラス
====================

目的
==========
主に移動境界において境界面の移動速度を計算する際に使われる. 

.. note::

    現在は三次精度まで対応している. デフォルトの設定は三次精度. 


理論
==========

変数 :math:`t` について非等間隔な離散化 :math:`t^{n+1} =t^{n} + \Delta t_{n} \; (n = 0,1,2,\dots)` を施すときの
関数 :math:`f^{n+1} = f(t^{n+1})` の微分 :math:`\left.\frac{df}{dt}\right |^{n+1}` について考える. 

一次精度
------------
:math:`\left.\frac{df}{dt}\right |^{n+1}` を :math:`f^{n+1},f^{n}` だけで表すことを考える. 
:math:`f^{n}` を :math:`f^{n+1}` を基にTaylor展開すると

.. math:: 
    :label: fn

    f^{n} = f^{n+1} - \left.\frac{df}{dt}\right |^{n+1} \Delta t_{n} 
    + \frac{1}{2} \left.\frac{d^{2}f}{dt^{2}}\right |^{n+1} (\Delta t_{n})^2
    + O((\Delta t_{n})^3)

となるから, 両辺を :math:`\Delta t_n` で割って整理すると

.. math:: 

    \left.\frac{df}{dt}\right |^{n+1} = \frac{f^{n+1}-f^{n}}{\Delta t_n} + O(\Delta t_n)

を得る. 打ち切り誤差は一次のオーダーなので, これは一次精度である. 

二次精度
-----------
ステンシルを更に拡げて, :math:`f^{n+1},f^n,f^{n-1}` で表すことを考える. 
:math:`f^{n-1}` のTaylor展開により次式を得る. 

.. math:: 
    :label: fn-1

    &f^{n-1} \\ 
    (&=f(t^{n+1}-\Delta t_n-\Delta t_{n-1})) \\
    &= f^{n+1} - \left.\frac{df}{dt}\right |^{n+1} (\Delta t_{n}+\Delta t_{n-1} )
    + \frac{1}{2} \left.\frac{d^{2}f}{dt^{2}}\right |^{n+1} (\Delta t_{n} + \Delta t_{n-1})^2 \\
    &+ O((\Delta t_{n} + \Delta t_{n-1})^3)
  
式 :eq:`fn` と :eq:`fn-1` から, 2階微分を消去する. :math:`(1)\times a - (2)\times b` として
2階微分を消去し1階微分を残すような :math:`a,b` は

.. math:: 
    
    a &= - \frac{\Delta t_n + \Delta t_{n-1}}{\Delta t_n \Delta t_{n-1}} \\
    b &= - \frac{\Delta t_n}{\Delta t_{n-1}(\Delta t_n+\Delta t_{n-1})}

であるので, これより

.. math:: 

    \left.\frac{df}{dt}\right |^{n+1}
    &= f^{n+1}\cdot \frac{2\Delta t_n + \Delta t_{n-1}}{\Delta t_n(\Delta t_n + \Delta t_{n-1})} \\
    &+ f^{n}\cdot \left(- \frac{\Delta t_n + \Delta t_{n-1}}{\Delta t_n \Delta t_{n-1}} \right) \\
    &+ f^{n-1}\cdot \left(\frac{\Delta t_n}{\Delta t_{n-1}(\Delta t_n + \Delta t_{n-1})}\right) \\
    &+ O\left(\frac{\Delta t_{n}^2(\Delta t_{n} + \Delta t_{n-1}) + \Delta t_{n}(\Delta t_n + \Delta t_{n-1})^2}{\Delta t_{n-1}}\right)

となる. とくに :math:`\Delta t_n = \Delta t_{n-1} = \Delta t` の場合は

.. math:: 

    \left.\frac{df}{dt}\right |^{n+1}
    = \frac{\frac{3}{2} f^{n+1} - 2f^{n} + \frac{1}{2}f^{n-1}}{\Delta t} + O(\Delta t^2)

となり, この差分が二次精度となることがわかる. 

三次精度以上について
-----------------------

三次精度以降も同様の手順を踏むことで高次差分を得られる. 

三次精度の場合は, :math:`f^{n-2}` の :math:`f^{n+1}` に関するTaylor展開

.. math:: 
    :label: fn-2

    &f^{n-2} \\ 
    &= f^{n+1} - \left.\frac{df}{dt}\right |^{n+1} (\Delta t_{n}+\Delta t_{n-1}+\Delta t_{n-2}) \\
    &+ \frac{1}{2} \left.\frac{d^{2}f}{dt^{2}}\right |^{n+1} (\Delta t_{n} + \Delta t_{n-1}+\Delta t_{n-2})^2 \\
    &+ \frac{1}{3!}\left.\frac{d^{3}f}{dt^{3}}\right |^{n+1}(\Delta t_{n} + \Delta t_{n-1}+\Delta t_{n-2})^3    

と先ほどの :eq:`fn`, :eq:`fn-1` を利用して, 2階微分と3階微分とを消去することにより得られる. 

:math:`(1)*a_1 + (2)*a_2 + (3)*a_3` により次の連立方程式を得る. 

.. math:: 
    a_1\Delta t_{n} + a_2(\Delta t_{n} + \Delta t_{n-1}) + a_3(\Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2}) &= -1 \\
    a_1\Delta t_{n}^{2} + a_2(\Delta t_{n} + \Delta t_{n-1})^{2} + a_3(\Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2})^{2} &= 0 \\
    a_1\Delta t_{n}^{3} + a_2(\Delta t_{n} + \Delta t_{n-1})^{2} + a_3(\Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2})^{3} &= 0 \\

あるいは行列表記で

.. math:: 
    \begin{bmatrix}
    \Delta t_{n} & \Delta t_{n} + \Delta t_{n-1} & \Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2}\\
    \Delta t_{n}^2 & (\Delta t_{n} + \Delta t_{n-1})^2 & (\Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2})^2\\
    \Delta t_{n}^3 & (\Delta t_{n} + \Delta t_{n-1})^3 & (\Delta t_{n} + \Delta t_{n-1} + \Delta t_{n-2})^3\\
    \end{bmatrix}
    \begin{bmatrix}
    a_1 \\ a_2 \\ a_3 \\
    \end{bmatrix}
    =
    \begin{bmatrix}
    -1\\0\\0
    \end{bmatrix}

のように書ける. これを解くと, 各係数は次のようになる. 
なお計算にはSympyを用いた. 

.. math:: 
    a_1 &= - \frac{\left(x + y\right) \left(x + y + z\right)}{x y \left(y + z\right)} \\
    a_2 &= \frac{x \left(x + y + z\right)}{y z \left(x + y\right)} \\
    a_3 &= - \frac{x \left(x + y\right)}{z \left(y + z\right) \left(x + y + z\right)} \\
    a_1 + a_2 + a_3 &= - \frac{3 x^{2} + 4 x y + 2 x z + y^{2} + y z}{x \left(x + y\right) \left(x + y + z\right)}

ただし :math:`x = \Delta t_n, y = \Delta t_{n-1}, z = \Delta t_{n-2}` とおいた. 
このとき, 三次精度後退差分は

.. math:: 
    \left.\frac{df}{dt}\right |^{n+1}
    \simeq -(a_1 + a_2 + a_3)f^{n+1} + a_1 f^{n} + a_2 f^{n-1} + a_3 f^{n-2}

で求められる. 

おそらく必要ないが, 一般化することは可能である. 
:math:`k+2` 個のデータ :math:`f^{n+1},f^{n},\dots,f^{n-k}\;(k<n)` を用いて :math:`k+1` 次精度の後退差分を求めるには, 

.. math:: 

    {\bf \Delta}{\bf a} = {\bf b}

なる線形方程式系を解いて :math:`{\bf a} = a_1,a_2,\dots, a_{k+1}` を求め

.. math:: 
    \left.\frac{df}{dt}\right |^{n+1}
    = \left(\sum_{l=1}^{k+1} a_{l}\right) f^{n+1} + \sum_{l=1}^{k+1} a_{l}f^{n+1-l}

により高次差分を求める. ただし,  

.. math:: 
    {\bf \Delta} =
    \begin{bmatrix}
    \Delta t_{n} & \Delta t_{n} + \Delta t_{n-1} & \cdots & \sum_{l=0}^{k} \Delta t_{n-l} \\
    \Delta t_{n}^2 & (\Delta t_{n} + \Delta t_{n-1})^2 & \cdots & \left(\sum_{l=0}^{k} \Delta t_{n-l}\right)^2 \\
    \vdots & \vdots & \ddots &\vdots \\
    \Delta t_{n}^{k+1} & (\Delta t_{n} + \Delta t_{n-1})^{k+1} & \cdots & \left(\sum_{l=0}^{k} \Delta t_{n-l}\right)^{k+1}
    \end{bmatrix},

    {\bf b}
    =
    \begin{bmatrix}
    -1\\0\\ \vdots \\0
    \end{bmatrix}

である. 

実装
==============
``backward_difference_t`` クラスに実装されている. 

ソルバでは, ``boundary_t`` のメンバとして保持されており, 移動境界の境界面速度を計算するために用いられている. 
なお算出に当たっては界面の重心ベクトル :math:`{\bf r}_{f}` を用いる. つまりベクトルの各成分に対して上記の後退差分を
適用することにより, 界面の移動速度ベクトルを計算する.