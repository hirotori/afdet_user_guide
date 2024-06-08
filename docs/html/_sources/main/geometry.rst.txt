=====================
幾何量の計算
=====================

有限体積法で流れ計算を実施する場合, セルの体積やセルの面の面積,法線ベクトルなどが必要となる.
ここではこれらの幾何量の計算方法についてまとめておく.
まず実際に用いられる幾何量計算のアルゴリズムについて説明する.
.. その後で実際のコードを例に関連付けて説明する.

面の表面積と重心
==================

本ソルバの場合, 面の形状は主に三角形(triangle)か四角形(quadlirateral)である.
多面体を扱えるようになると, さらに別の多角形も有り得る. 

面積の計算
------------------------

三角形面の面積はベクトルの外積を利用する. 

.. math:: 
    :label: eq_tri_surface_area

    \boldsymbol{S} = \frac{1}{2} \left(\boldsymbol{r}_{2}-\boldsymbol{r}_{1} \right) \times \left(\boldsymbol{r}_{3}-\boldsymbol{r}_{1} \right) 


.. figure:: /images/face_area.png

    三角形の面積(ベクトル).

四角形など一般の形状については以下の座標法(または靴紐の公式, shoelace formula)を利用する事が出来る.
面を構成する頂点を反時計回りに並べるとき, 面積は次式で計算出来る.

.. math:: 

    \boldsymbol{S} = \frac{1}{2} \sum_{v = 1}^{N_{v}} \boldsymbol{r}_{v} \times \boldsymbol{r}_{v+1}
   
ここで, :math:`\boldsymbol{r}_{N_{v}+1}=\boldsymbol{r}_{1}` である. 
とくに四角形の場合は, 上式を適用した上で式変形を施し, 

.. math:: 

    \boldsymbol{S} = \frac{1}{2} \left(\boldsymbol{r}_{3}-\boldsymbol{r}_{1} \right) \times \left(\boldsymbol{r}_{4}-\boldsymbol{r}_{2} \right)

として計算する. ``grid_t`` クラスのメンバ ``face_areas(1:face_count)`` に面の面積が格納される.

.. figure:: /images/quadlilateral_surface.png

    四角形の面積の公式.

.. _重心の計算:

重心の計算
-----------------------------

次に重心(centroid)の計算について述べる. 

.. note:: 

    geometric center と centorid の用語の使い分けが曖昧. Wikipediaだと geometric center と centorid は同一視されている.
    しかしMoukalledの本だと, geometric center は図形を構成する頂点の算術平均で, centroid は別物扱いされている.
    とりあえずMoukalledの本に沿って用語を扱う.

三角形の場合, 重心は幾何図心と一致する. すなわち,

.. math:: 
    :label: eq_tri_centroid

    \boldsymbol{r}_{C} = \boldsymbol{r}_{G} = \frac{\boldsymbol{r}_{1} + \boldsymbol{r}_{2} + \boldsymbol{r}_{3}}{3}


四角形の場合は, 2つの三角形に分割し, 以下の公式により求める.

.. math:: 

    \boldsymbol{r}_{C} = \frac{\sum_{t=1}^{2} \boldsymbol{r}_{G}^{t}\cdot S}{S}

ここで :math:`\boldsymbol{r}_{G}^{t}` は分割した各三角形 :math:`t` の図心で, 式 :eq:`eq_tri_centroid`  より計算する.  
また :math:`S=|\boldsymbol{S}|` は分割した三角形の面積で 式 :eq:`eq_tri_surface_area` より計算する.

``grid_t`` クラスのメンバ ``face_centers(1:3,1:face_count)`` に面の重心が格納される.

セルの体積と重心
==================

体積の計算
--------------------

3次元セルの体積はGaussの発散定理を利用して計算する.
ベクトル値関数 :math:`\boldsymbol{f}(\boldsymbol{x})=(f_{1}(\boldsymbol{x}),f_{2}(\boldsymbol{x}),f_{3}(\boldsymbol{x}))^{T}` 
に対してガウスの発散定理は

.. math:: 

    \int_{V}\nabla \cdot \boldsymbol{f} dV=\int_{S} \boldsymbol{n} \cdot \boldsymbol{f} dS

で表される. ここで :math:`V` は検査体積, :math:`S` は検査表面, :math:`\boldsymbol{n}` は外向き法線ベクトルである.
また, :math:`\nabla \cdot \boldsymbol{f} = \frac{\partial f_{i}}{\partial x_{i}}` である. 
ベクトル値関数として :math:`f_{i}=x_{i}` を取ると,

.. math:: 

    \int_{V}\nabla \cdot \boldsymbol{x}dV=\int_{S}\boldsymbol{n} \cdot \boldsymbol{f}dS

ここで, 左辺は :math:`\frac{\partial x_{i}}{\partial x_{i}}=3` より

.. math:: 

    \int_{V}\frac{\partial x_{i}}{\partial x_{i}}dV=3\int_{V}dV=3V

なので, 

.. math:: 

    V=\frac{1}{3}\int_{S}n_{i}x_{i}dS

と計算出来る. 右辺の積分を,重心点で近似すると,

.. math:: 

    V=\frac{1}{3}\int_{S}n_{i}x_{i}dS \simeq \frac{1}{3} \sum_{f \sim nb(C)} \boldsymbol{n}_{f} \cdot \boldsymbol{x}_{f}S_{f}

を得る. これが体積を求める近似式である. この方法は任意の図形に適用出来る. 
四角形などの面がねじれて平面でなくなる場合は,三角形に分割してそれぞれの三角形についての和を取れば良い.
``grid_t`` クラスのメンバ ``cell_volumes(1:cell_count)`` に格納される.

重心の計算
-------------------

重心は以下の公式から計算される. 導出はWang [1]_ を参照.

.. math:: 
    :label: eq_cell_centroid

    \boldsymbol{r}_{C} = \frac{3}{4}\frac{\sum_{f \sim nb(C)}(\boldsymbol{n}_{f}\cdot \boldsymbol{r}_{f})\boldsymbol{r}_{f}S_{f}}{\sum_{f \sim nb(C)}\boldsymbol{n}_{f}\cdot \boldsymbol{r}_{f}S_{f}}

.. warning:: 

    本ソルバはこの方法を採用しているのだが, **重心がセルから飛び出すケースがある.**
    四角形面が平面から外れている(ねじれている)場合に起こりやすい.
    面を三角形分割して対処するか, あるいは飛び出した場合にそのセルだけ幾何中心(即ちセルの節点座標の和を節点数で割った平均)で代用するなどの対策が必要である.
    本ソルバは後者を採用している. 詳細は ``grid.f90`` のプリプロセッサマクロ ``PRISM_CENTER_CORRECTIONE`` を参照.

``grid_t`` クラスのメンバ ``cell_centers(1:3,1:cell_count2)`` に格納される. 仮想セルに対してもセル重心を, 
面に関して鏡面対象となる位置に定義する. (:numref:`fig_boundary_and_ghost_cells` 参照)

その他
======================

.. _面ｰセル間距離 :

面-セル間距離
-----------------------
``grid_t`` クラスのメンバ ``face_cell_disances(1:2,1:face_count)`` は, 面からセル重心への距離を格納する.
第1次元は面から見てどちらのセルかを意味する. (:ref:`面からセルへ` のアクセス方法と同じ.)
セル重心座標を :math:`{\bf r}_{C}` , 面の重心を :math:`{\bf r}_{f}` ,  面の単位法線を :math:`{\bf n}_{f}` とおくと,
以下のように表される.

.. math:: 

    d_{Cf} = | ({\bf r}_{f} - {\bf r}_{C}) \cdot {\bf n}_{f} |


面ｰセル間距離は主に物理量の面への内挿を計算する際の重み (weighting factor) として用いられる.

.. figure:: /images/face_cell_dist.png

    セル重心と, 面との間の距離.

**References**

.. [1] Z.J. Wang, Improved Formulation for Geometric Properties of Arbitrary Polyhedra, AIAA Journal. 37 (1999) 1326–1327. https://doi.org/10.2514/2.604.
