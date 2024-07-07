============================
格子の変形
============================

空間格子の移動や変形は, 汎用ケースクラス ``case_commom_t`` を継承したカスタムケースクラスのメソッド ``on_transform_grid`` を呼び出し, 
格子の座標を更新する. ``on_transform_grid`` は以下のような引数仕様を持つ.

.. code:: Fortran

    subroutine on_transform_grid(this, grid, fluid, xyz)
        class(your_case_t), intent(inout) :: this
        type(grid_t), intent(in) :: grid
        type(fluid_t), intent(in) :: fluid
        real(WP_), intent(inout) :: xyz(:,:)
    
    end subroutine


変更可能なのは ``grid_t`` オブジェクトではなく, その中のメンバである格子の座標 ``xyz`` である. 
上のメソッドは, 座標配列 ``xyz`` を ``grid`` とは別の仮引数として与えることで, ``grid_t`` の他のメンバ変数を変更しないように配慮されている.

強制変位の例題 (ピストン流)
==============================

例として, 円管ピストン流を格子変形を使って再現してみよう. 直径 :math:`D`, 長さ :math:`L` の円管を考える. 長手方向を :math:`x` 軸とし, 
円管の中心が :math:`x` 軸を通るように座標をとる. 円管の左端を速度 :math:`V` で動かし, 右端を固定して, 円管を徐々に圧縮する. 
このときの円管内の流れを移動格子有限体積法により計算する. 

強制変位(1自由度)
------------------------------
ばピストン運動のような1自由度の移動(直線上の運動)の場合は,バネ格子をわざわざ利用せずとも,
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

円管の左端が :math:`x_{0}=x_L` , 右端が :math:`x_{0}=x_R\;(x_R > x_L)` にあるとする．
左端を変位 :math:`l(t)=V*\Delta t` で動かし，右端を固定するとき :math:`x_L < x < x_R` に存在する節点を動かすための
移動式の係数 :math:`a(t), b(t)` を求める.

:math:`f(x_{a,0},t) = A(t)x_{a,0} + B(t)` とおく. 左端の節点 :math:`a` の時刻 :math:`t` における **変位** は

.. math:: 

    x_{a}(t)-x_{a,0} = l(t)

に等しい. (絶対位置ではなく変位であることに注意.) よって

.. math:: 

    A(t)x_L + B(t) = l(t)

が成り立つ. 同様に右端についても

.. math:: 

    A(t)x_R + B(t) = 0

が成り立つので, 連立して :math:`A,B` を求めると,

.. math:: 

    x(t) - x_{0} = \frac{l(t)}{x_L - x_R}x_{0} - \frac{l(t)}{x_L - x_R} x_R

となる. (下添え字 :math:`a` は省略.)

これを ``on_transform_grid`` に実装する. 

.. code:: Fortran
    
    subroutine on_transform_grid(this, grid, fluid, xyz)
        class(case_piston_t), intent(inout) :: this
        type(grid_t), intent(in) :: grid
        type(fluid_t), intent(in) :: fluid
        real(WP_), intent(inout) :: xyz(:,:)
        
        real(WP_) dt
        real(WP_) a, b, dx
        
        dt = this%get_dt()        
        a = this%left_
        b = this%right_
        dx = PistonSpeed_ * dt
        
        xyz(1,:) = b + (xyz(1,:) - b) * (b - a - dx) / (b - a)
        
        this%left_ = this%left_ + dx
        
        print *, "current length : ", this%right_ - this%left_
    end subroutine




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
