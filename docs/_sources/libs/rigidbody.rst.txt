===================
剛体の運動(弱連成)
===================

理論
==============

周囲流体の力を受けて運動する剛体の運動を考える. 

並進運動
--------------

物体の質量を :math:`M` , 重心の位置ベクトルを :math:`{\bf r}_{G}` とおく. 
Newtonの第2法則に従うとすれば, 物体の運動は次のような微分方程式で記述される. 

.. math:: 
    :label: Newtons_2nd_law

    M \ddot{{\bf r}_{G}} = {\bf F} + M{\bf g}


ここで, :math:`{\bf g}` は重力加速度である. 座標系のどの向きを重力が働く方向とするかによって, 
:math:`{\bf g}` の向きは変わる. 例えば :math:`z` を重力の働く方向と設定するならば, :math:`{\bf g} = g {\bf e}_{z}` である. 
ただし :math:`{\bf e}_{z}` は単位ベクトルで, :math:`g = \left\| {\bf g} \right\| \simeq 9.806 {\rm [m/s]}` である. 

.. note:: 重力加速度の大きさについて

    後述するが大きさはユーザが設定しなければならない. 




のちの離散化のため, :eq:`Newtons_2nd_law` を次のように2段に分けて書く. 

.. math:: 
    :label: system_newton

    M \dot{{\bf v}} &= {\bf F} + M{\bf g} \\
    \dot{{\bf r}_{G}} &= {\bf v}


回転運動
---------------

剛体のある点周りの運動はつぎのオイラー方程式に従う. 空間に固定された慣性座標系を :math:`(x,y,z)` で表し, 
これをグローバル座標系(あるいはワールド座標系)とよぶ. 
また, 剛体の重心を原点にとりその慣性主軸 :math:`(\xi,\eta,\zeta)`  を剛体と共に回転するあらたな座標系とし, 
これをローカル座標系と呼ぶことにする. 

慣性座標系で回転運動を表すと, 慣性モーメントと慣性主軸の時間微分が含まれるため厄介である. そこで, 数値計算では
Eulerの運動方程式を用いることでこの欠点を避ける. 

剛体の角速度 :math:`{\boldsymbol \omega}` の :math:`\xi,\eta,\zeta` 方向成分を :math:`\omega_{\xi},\omega_{\eta},\omega_{\zeta}` 
とし, トルク :math:`{\bf T}` の :math:`\xi,\eta,\zeta` 方向成分を :math:`T_{\xi},T_{\eta},T_{\zeta}` とすると,  

.. math:: 
    :label: Euler_eqs

    I_{\xi} \frac{d \omega_{\xi}}{dt} - \left( I_{\eta} - I_{\zeta} \right) \omega_{\eta} \omega_{\zeta} = T_{\xi} \\
    I_{\eta} \frac{d \omega_{\eta}}{dt} - \left( I_{\zeta} - I_{\xi} \right) \omega_{\zeta} \omega_{\xi} = T_{\eta} \\
    I_{\zeta} \frac{d \omega_{\zeta}}{dt} - \left( I_{\xi} - I_{\eta} \right) \omega_{\xi} \omega_{\eta} = T_{\zeta} \\


で表される. ただし :math:`I_{\xi},I_{\eta},I_{\zeta}` は :math:`\xi,\eta,\zeta` 方向の主慣性モーメントである. 


主慣性モーメント
^^^^^^^^^^^^^^^^^^^

:eq:`Euler_eqs` より回転角速度を求めるためには, 慣性主軸と主慣性モーメントを求める必要がある. 
慣性モーメントはその定義から実対称行列である. したがって直交行列 :math:`P` によって対角化することが出来る. 

.. math:: 

    P^{-1}IP = \Lambda = 
    \begin{bmatrix}
    \lambda_{1} & 0 & 0 \\
    0 & \lambda_{2} & 0 \\
    0 & 0 & \lambda_{3}
    \end{bmatrix}


右辺の :math:`\lambda_{i}\: (i = 1,2,3)` が主慣性モーメントである. 
また直交行列 :math:`P` は座標変換の行列である. 例えばグローバル座標系の角運動量

.. math:: 

    L = I \boldsymbol{\omega}

を座標変換する. 慣性主軸から測った角速度ベクトルを :math:`\boldsymbol{\omega}'` とおく. 
両者に :math:`\boldsymbol{\omega} = P \boldsymbol{\omega}'` の関係があるとすると, 

.. math:: 

    L = IP\boldsymbol{\omega}'

両辺に :math:`P^{-1}` を左から掛けると, 

.. math:: 

    P^{-1}L = \Lambda \boldsymbol{\omega}'

となって, この座標系では慣性テンソルが対角化されていることがわかる. 

したがって, グローバル座標系から慣性主軸への変換は, 左から逆行列 :math:`P^{-1} = P^{T}` を
掛けることで求められる.  


クォータニオン
--------------

式 :eq:`Euler_eqs` は剛体と共に回転する座標系における成分の時間変化を示すため, ワールド座標系からみた運動へと変換する必要がある. 
変換方法には主に2つある. 

* オイラー角
* クォータ二オン

しかしながら, オイラー角はジンバルロックが生じる可能性があり, 例えば飛行機の宙返り運動などを再現する場合には
計算が破綻する恐れがある. そこで本ソルバではクォータ二オン(quaternion, 四元数)を採用する. 

離散化
================

数値計算を行うために離散化を施す必要がある. 並進運動の場合について考える. 運動方程式を
時間間隔 :math:`[t^{n} \: t^{n+1}]\:(t^{n}=n\Delta t,\;n=0,1,2,\dots)` で積分すると

.. math:: 

    M \int_{t^{n}}^{t^{n+1}} \dot{{\bf v}} dt &= \int_{t^{n}}^{t^{n+1}} \left({\bf F} + M{\bf g}\right) dt \\
    \int_{t^{n}}^{t^{n+1}} \dot{{\bf r}} dt &= \int_{t^{n}}^{t^{n+1}} {\bf v} dt

を得る. 

左辺の積分は次のようになる. 

.. math:: 

    \int_{t^{n}}^{t^{n+1}} \dot{{\bf v}} dt &\simeq {\bf v}^{n+1} - {\bf v}^{n} \\
    \int_{t^{n}}^{t^{n+1}} \dot{{\bf r}} dt &\simeq {\bf r}^{n+1} - {\bf r}^{n}

上付き文字はどの時間段階で評価するのかを表す. 
また右辺の評価については, 第一式は前進オイラー法, 第二式はクランク･ニコルソン法を適用する. 

.. math:: 

    \int_{t^{n}}^{t^{n+1}} \left({\bf F} + M{\bf g}\right) dt &\simeq \left({\bf F} + M{\bf g}\right)^{n} \Delta t 
     = {\bf F}^{n} \Delta t + M{\bf g} \Delta t\\
    \int_{t^{n}}^{t^{n+1}} {\bf v} dt &\simeq \frac{{\bf v}^{n} + {\bf v}^{n+1}}{2} \Delta t

これらを組み合わせて整理すると, 新しい時刻での位置と速度は次式で求められる. 

.. math:: 

    {\bf v}^{n+1} &= {\bf v}^{n} + \frac{{\bf F}^{n}}{M} \Delta t + {\bf g}\Delta t \\
    {\bf x}^{n+1} &= {\bf x}^{n} + \frac{{\bf v}^{n} + {\bf v}^{n+1}}{2} \Delta t

実装
==================

剛体の運動は ``Rigidbody_t`` クラスが受け持つ. 

事前準備
-----------------

* メンバ変数の ``LocalGravity`` で重力を指定する. 不要な場合は指定しない(デフォルトでゼロ).
* メンバ変数の ``CenterOfMass`` に初期重心座標を入力する. 
* 回転運動をする場合, ``SetInertiaTensor`` でグローバル座標系の慣性テンソルを入力する. 

並進運動
------------------

並進運動の場合は以下の流れに従う. 

1. 既知の流れ場において, 物体表面に働く流体力の合力 :math:`{\bf F}^{n}` を求める
2. 新しい時間段階での速度 :math:`{\bf v}^{n+1}` を求める. 
3. 新しい重心の位置ベクトル :math:`{\bf r}_{G}^{n+1}` を求める. 

上記のうち2.と3.はまとめてメンバ関数 ``update`` が担う. 1.に関しては, 
``fluid.f90`` 内のサブルーチン ``calc_fluid_force_and_torque`` により合力を求め, 
``Rigidbody_t`` クラスのメンバ関数 ``addForce`` を呼び出してクラスに合力を登録する. 

なお格子を動かすのはユーザの役目である. 方法は主に2つ. 

* 1ステップ前の位置ベクトルをクラス外に記憶しておき, 新しく計算された位置との差分から変位を求める. 
* 物体の重心からの各点の相対ベクトルを記憶しておき, 新しい座標を重心の位置ベクトルとの和で計算する. 

上記2つは実質的に同じことをしている. 剛体中のある質点 :math:`i` の座標を :math:`{\bf r}_{i}` とすると, 
剛体内の任意の2点間距離は運動の間変化しないから, 
**回転を含まない運動** に限っては重心から見た質点の相対ベクトル

.. math:: 

    {\boldsymbol \rho}_i = {\bf r}_{i} - {\bf r}_{G}

は時間によらず一定である. そのため, 剛体上の任意の格子点の :math:`t=t^{n+1}` における新しい位置は

.. math:: 

    {\bf r}_{i}^{n+1} &= {\bf r}_{G}^{n+1} + \boldsymbol{\rho}_i \\
    &= {\bf r}_{G}^{n+1} - {\bf r}_{G}^{n} + {\bf r}_{i}^{n} \\
    &= \Delta {\bf r}_{G} + {\bf r}_{i}^{n}

と書ける. 第一式は後者にあたり, そこから前者が導かれる(第3式).


回転運動
--------------

``Rigidbody_t`` クラスが保持するのは剛体の回転行列である. 物体を回転させるためにはこの回転行列を
用いてユーザが処理を書かなければならない. 回転行列 ``rotation`` がクォータ二オンなので, 
ベクトルの回転は2通り有り得る. 

* ``rotation`` を ``ToRotationMatrix`` で回転行列に変換する
* ``quaternion_t`` のメソッド ``rotate`` を呼び出す. 

たぶん2つ目のやり方が確実. ただし ``rotate`` の引数に与えるベクトルは **初期座標** で無ければならない. 

参考
=======

* 原島 鮮, 力学Ⅰ―質点･剛体の力学―
* 三宅 敏恒, 入門線形代数
* J.L.Meriam, L.G.Kraige, J.N.Bolton, Engineering Mechanics Volume 2 Dynamics 8th edition, Wiley