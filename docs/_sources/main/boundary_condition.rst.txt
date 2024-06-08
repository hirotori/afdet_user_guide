=========
境界条件
=========

境界条件に関するクラスとその関係性, 実装について. 

理論
============

本ソルバでは, 境界条件に仮想セル(ゴーストセル, ダミーセル)を用いる (:numref:`fig_boundary_and_ghost_cells`).  
各種の境界条件は, ゴーストセルに外挿されることで反映される. 

ゴーストセルは, その中心が境界セルのセル中心と面を挟んで対称となるように設定される. 
境界セルとゴーストセルのセル中心座標をそれぞれ :math:`{\bf r}_{C}, {\bf r}_{G}` ,境界面の重心座標を
:math:`{\bf r}_{f}` とおく. 境界面の物理量 :math:`q_{f}` は次のような線形内挿により表現される. 

.. math:: 
    :label: eq_boundary_value_interpol

    q_{f} = \frac{d_{fC}\cdot q_{G} + d_{Gf}\cdot q_{C}}{d_{fC}+d_{Gf}}

    d_{fC} &= | \left({\bf r}_{C} - {\bf r}_{f} \right)\cdot {\bf n} | \\
    d_{Gf} &= | \left({\bf r}_{f} - {\bf r}_{G} \right)\cdot {\bf n} | \\


:math:`d_{fC},d_{Gf}` はそれぞれ面-セル重心間の距離である (:ref:`面ｰセル間距離` 参照.) .
一方, 境界面の物理量が既知の時に, ゴーストセルの値を更新するためには上式を変形して

.. math:: 
    :label: eq_ghost_value_extrapol

    q_{G} = \frac{(d_{fC}+d_{Gf})q_{f} - d_{Gf}\cdot q_{C}}{d_{fC}}

とする.
固定値境界では基本的にこの内挿関係に従って境界値, ゴーストセルの値を決定する. 
``boundary_condition.f90`` には境界面およびゴーストセルの値を決定する関数が用意されている.
式 :eq:`eq_boundary_value_interpol` に基づいて境界値を決めるのが ``calc_wall_value`` . 
逆に式 :eq:`eq_ghost_value_extrapol` よりゴーストセルの値を求めるのが ``calc_ghost_value`` である.  


種類
=========

基本的な境界条件について見てみる. 境界面の速度を :math:`{\bf u}_{bnd.}` とおく. 

流入条件
----------
一様流入条件の場合, **セル境界面** に流入速度を与える. つまり, 

.. math:: 

    {\bf u}_{bnd.} = {\bf u}_{in}

したがってゴーストセルの値は

.. math:: 

    {\bf u}_{G} = \frac{{\bf u}_{in}\cdot \left( d_{fC}+d_{Gf} \right) - {\bf u}_{C}\cdot d_{Gf}}{d_{fC}}

で与えられる. 


流出条件
---------

勾配ゼロ規定の場合は次のように考える. 与えられる条件はある物理量の **境界面に垂直な方向微分がゼロ** という条件なので, 

.. math:: 

    \frac{\partial q}{\partial n} = 0

である. いま, セル中心間を結ぶベクトルと法線は平行なので, この方向微分は

.. math:: 

    \frac{\partial q}{\partial n} \simeq \frac{q_{G}-q_{C}}{d_{CG}}

と近似出来る. それゆえ, 2つを組み合わせると

.. math:: 

    q_{G} = q_{C}

なる式を得る. 速度の場合も同様に, 成分毎に考えればよいので, 

.. math:: 

    {\bf u}_{G} = {\bf u}_{C}

を得る.
なお, 自由流出入条件( ``free_stream`` )の場合, 流速の向きで流速規定と勾配ゼロ規定を切り替える. 

滑り無し壁条件
------------------

これは粘性流体の粘着条件であり, 適用する境界面 :math:`\Gamma_{w}` 上の速度がその境界の移動速度 :math:`{\bf v}_{wall}` に等しいとして
与える. 

.. math:: 

    {\bf u} = {\bf v}_{wall} \quad on \quad \Gamma_{w}

したがって, 仮想セルの速度は 式 :eq:`eq_ghost_value_extrapol` より計算する. 
滑り無し条件は流入条件と同様に,境界の値を固定する条件である. 両者は壁が静止しているときは全く同じである.
しかし, 壁が移動する場合は壁の移動速度に基づいて計算されるので, 両者はこの点で異なる.

.. note:: 

    指摘したように, 滑り無し条件を与える壁が全く動かない場合は, 
    滑り無し条件を与えても速度0の速度固定条件を与えても同じである. 

なお, :math:`{\bf v}_{wall}` は境界面上の移動量から計算する. デフォルトは三次精度後退差分である. 
詳細は :doc:`/libs/backward_diff` を参照.


滑り壁条件
------------------

滑り条件では流体が壁面を滑る. したがって, 壁面に摩擦応力は発生しない. また, 流体粒子は壁面に沿って滑ることは出来るが,
壁面を貫通することは出来ないので, 流体の壁面に垂直な方向の速度成分は壁の移動速度の法線方向成分と一致しなければならない [1]_ .
壁面の移動速度を :math:`{\bf v}_{wall}` , 壁の単位接線ベクトルを :math:`{\bf t}` とおくと, 
壁面上の流体の法線方向速度成分は

.. math:: 

    (u_{bnd.})_{n}={\bf u}_{bnd}\cdot {\bf n} = {\bf v}_{wall}\cdot {\bf n}

また, 接線方向速度は壁から第1点のセル速度を参照して, 

.. math:: 

    (u_{bnd.})_{t}{\bf t} = {\bf u}_{bnd}\cdot {\bf t} = {\bf u}_{C} - ({\bf u}_{C} \cdot {\bf n}){\bf n}

とする. よって, 境界面の速度は

.. math:: 

    {\bf u}_{bnd.} &= (u_{bnd.})_{n}{\bf n} + (u_{bnd.})_{t}{\bf t} \\
    &= {\bf u}_{C} + \left\{ ({\bf v}_{wall} - {\bf u}_{C})\cdot {\bf n} \right\} {\bf n}

である. 

.. figure:: /images/slip_boundary.png

    滑り境界条件


Poisson方程式への境界条件の組み込み
====================================

.. warning::

    バージョンによっては正確な記述では無い可能性. 

非圧縮性流体のシミュレーションにおいては, 圧力に関するPoisson方程式を解く必要がある. 
本ソルバでは, この方程式を :math:`A{\bf x}={\bf b}` の形式で解く. ここで :math:`{\bf x}` は
求めるべきセル中心圧力を成分にもつベクトルである. 

ゴーストセルを使う都合上, 境界セルに対する定式化は内部セルと同様である. それゆえ, 解ベクトル :math:`{\bf x}` には
内部セルの変数 :math:`p_{C}\: (C=1,2,\dots ,n_{cellcount})` のみならずゴーストセルの変数
:math:`p_{G}\: (G=n_{cellcount}+1,n_{cellcount}+2,\dots ,n_{cellcount2})`  も含む. したがって, 
システム方程式 :math:`A{\bf x}={\bf b}` を完全なものにするには, 内部セルに対する式

.. math:: 

    a_{C}p_{C} + \sum_{f}^{NB(C)} a_{f} p_{f} = b_{C}


のみならず, ゴーストセルに対する式も組み込まなければならない. 

.. math:: 

    a_{G}p_{G} + \sum_{f}^{NB(G)} a_{f} p_{f} = b_{G}

ゴーストセルに対する離散式の係数 :math:`a_{G}, a_{f}, b_{G}` は境界条件から定まる. 
通常, ゴーストセル1個あたりに隣接する内部セル(=境界セル)は1つだけであるから, :math:`NB(G) = 1` である. 
しかし, スライド境界面の場合は隣接するセルが1つとは限らない. このような場合にも対処するため, 本ソルバでは
ゴーストセルとそれに隣接するセルとの関係情報を隣接行列を用いて保持している.   

圧力固定
------------
例えばスライド面ではない普通の境界面で :math:`p_{b}` の圧力固定境界を与える場合, ゴーストセルの値は

.. math:: 

    p_{G} = p_{b}

で与えられる. 速度の境界条件と異なり, ゴーストセルに直接値を与えていることに注意する. したがって, この場合の係数は

.. math:: 

    a_{G} &= 1 \\
    a_{f} &= 0 \\
    b_{G} &= p_{b}

となる. 

圧力勾配ゼロ
-------------------
同様に, 圧力勾配ゼロの場合は

.. math:: 

    p_{G} - p_{C} = 0

であるから

.. math:: 

    a_{G} &= 1 \\
    a_{f=C} &= -1 \\
    b_{G} &= 0

となる. 

実装と使い方
===============
計算を実施する場合や, 新しく境界条件を設定する場合については, 境界条件クラスの作成方法だけわかればOK.


境界ゾーンクラス
-----------------
境界条件は流れを解く領域の境界(boundary)ゾーンに与える.
例えば, 入口に一様流が流れ込み, 出口で流出する円管ポアズイユ流を解く場合, 3つの境界ゾーン― 流入面, 流出面, 壁面 ―を考慮する. 
それぞれの境界ゾーンに適切な境界条件(一様流入, 流出, 滑り無し条件)を与えることで正しく問題を解くことが出来る.
それぞれの境界ゾーンは2次元図形の集まりである. 計算領域を四面体格子で分割した場合は，得られる境界ゾーンは
全て三角形で構成されている. 

各境界ゾーンの面へのアクセスは ``grid.f90`` の ``boundary_offsets`` および ``boundary_faces``
の2つの配列から行う. 詳細は :doc:`/main/fvm_mesh_data` を参照のこと.

各境界ゾーンに指定した境界条件を適用する実際の操作は, ``boundary_t`` クラスで行われる. 

境界条件クラス
--------------

境界条件は ``boundary_condition.f90`` で定義される. ここでは
境界条件を設定するための基底クラス ``patch_field_t`` が定義されている. 

.. code-block:: Fortran

    type, public, abstract :: patch_field_t
    contains
        procedure(IEvaluate), public, deferred :: evaluate 
        
        procedure, public :: fixes_value => fixes_value_false_
    end type
    
    abstract interface
        pure subroutine IEvaluate(this, q, face_velocity, face_center, face_normal, distance, prim_count, ghost_q)
            import patch_field_t
            import WP_
            class(patch_field_t), intent(inout) :: this
            real(WP_), intent(in) :: q(:), face_velocity(3), face_center(3), face_normal(3), distance(2)
            integer, intent(in) :: prim_count
            real(WP_), intent(out) :: ghost_q(:)
        end subroutine  
                
    end interface

プロシージャ ``evaluate`` は仮想セルを更新する処理であり, ``fixes_value`` は境界条件が固定値境界か勾配規定かを
判定する処理である. ``evaluate`` の引数は, 抽象クラスのインターフェース宣言 ``abstract interface ~ end interface`` を読めば分かるように, 
**ある1つの境界面に対して条件を適用する** 仕様になっている. したがって, あるゾーンの全ての境界面に対して, この関数が毎回呼び出される.
また, この関数は毎時間ステップ呼び出されるので, 例えば流量が時間的に変化する流入条件のようなことも原理的には可能である.

具体的な境界条件クラスはこれを継承して実装される. 例えば固定値境界と勾配ゼロ規定については既に用意されている. 

.. code-block:: Fortran

    !> @brief 固定値流入の境界条件を設定する。
    type, public, abstract, extends(patch_field_t) :: fixed_value_patch_t
    contains
        procedure, public :: fixes_value => fixes_value_true_
    end type
    
    
    !> @brief 勾配0のノイマン境界条件を設定する。
    type, public, extends(patch_field_t) :: zero_gradient_patch_t
    contains
        procedure :: evaluate => bc_zero_gradient_
    end type

さらに拡張したい場合はこれらのクラスを継承して新しい子クラスを作る. 

壁面については別途ことなる基底クラスが用意されている. 

.. code-block:: Fortran

    type, public, abstract, extends(patch_field_t) :: wall_patch_t

        integer, public :: velocity_type = 0  !< 速度の種類。
                                           !! @details 0ならwall_velocityが使用される。1なら一様速度が使用される。
        real(WP_), public :: uniform_velocity(3) = 0 !< 平行速度。壁の初期速度としても利用する。
        real(WP_), public :: origin(3) = [0,0,0]     !< 回転原点。
        real(WP_), public :: axis(3) = [0,0,0]       !< 単位方向ベクトル。
        real(WP_), public :: omega = 0               !< 角速度。 [rad/s]
        
        !real(WP_), allocatable, public :: wall_velocity(:,:) !< 各面の速度。
        !                                                     !! @note 使用者が適切なサイズに割り当てる必要がある。
    contains
        procedure, public, non_overridable :: get_velocity => get_velocity_wall_patch_
    end type

必要ならこれらを継承すれば良い. 

ゾーンとの対応付け
------------------

.. note:: 

    このページの存続について. ガイドとするにはやや細かすぎるかもしれない.

境界条件はそれを適用する解析領域の境界面(ゾーンと呼ぶ)と関連付けられる. 例えば流入面に対応するゾーンに対しては速度固定, 
圧力ノイマンなどの条件を与えるだろう. 

境界条件はコンフィグファイル ``case_setting.txt`` でゾーン毎に対応する番号を指定することで適用出来る. 
各ゾーン番号とその境界条件番号, および与える境界値は構造体 ``case_setting_t`` のメンバ ``zone_t`` が管理している. 
コンフィグファイルで指定された番号に対応する境界条件クラス ``patch_field_t`` とゾーンとを結びつける役割は, ``case_setting.f90`` の関数
``create_bc_default`` が担う. 返値は対応する境界条件クラスのポインタである. 

境界条件とゾーンとの対応付けはケースクラス ``case_common_t`` で行われる. 一部抜粋する. 

.. code-block:: Fortran

    subroutine setup_bc_(this, fluid, grid, zone_count, is_restart)
        class(case_common_t), intent(inout) :: this
        class(fluid_t), intent(in) :: fluid
        class(grid_t), intent(in) :: grid
        integer, intent(in) :: zone_count
        logical, intent(in) :: is_restart
   
        integer i, bounds(2,zone_count), bc_type, ids(zone_count), bc_types(zone_count), q_size
        real(WP_) uvwp(4), q(10)
        class(patch_field_t), allocatable :: bc        
        class(patch_field_t), pointer :: ptr

        bounds(1,:) = grid%boundary_offsets(1:zone_count)
        bounds(2,:) = grid%boundary_offsets(2:zone_count+1) - 1

        !中略

        allocate(this%bc_pressure_(zone_count))
        allocate(this%bc_velocity_(zone_count))
        
        do i = 1, zone_count
            uvwp = this%case_setting_%zones(i)%ruvwp(2:5)
            bc_type = this%case_setting_%zones(i)%type_u
            ptr => create_bc_impl_(this, bc_type, uvwp(1:3), fluid%heat_capacity_ratio)
            call this%bc_velocity_(i)%set_bc(ptr, bounds(:,i))

            bc_type = this%case_setting_%zones(i)%type_p
            ptr => create_bc_impl_(this, bc_type, uvwp(4:4), fluid%heat_capacity_ratio)
            call this%bc_pressure_(i)%set_bc(ptr, bounds(:,i))
        end do

ループ内で, 各ゾーンの境界値 ``ruvwp`` と境界条件番号 ``type_u, type_b`` を用いて ``create_bc_impl_`` にて
対応する境界条件クラスを取得し, ``bc_holder_t`` クラスのインスタンス ``bc_velocity_, bc_pressure_`` に格納する. 
なお ``bounds(:,i)`` は各ゾーン ``i`` の面と関連付けられたオフセット配列である. 

境界条件の適用
---------------
上の処理で対応付けは完了した. 実際に境界条件を適用する(=境界面の値と境界セルの値から仮想セルへ外挿する)ためには, 面番号や面の幾何情報などが必要である. 
境界条件を適用するために必要なこれらの情報を取り扱うクラスが ``boundary_t`` である. ``case_common_t`` クラスのメンバとして保持されている.


.. code-block:: Fortran

    type, public :: boundary_t       
        integer, private, allocatable :: face2cell_(:,:)        
 
        type(bc_face_t), public, allocatable :: faces(:)       
        
        ! patch の中をdo版にしたとkは以下を使う。
        !integer, private, allocatable :: cells_(:,:) ! 1:内部セル、2:ゴーストセル。
        !real(WP_), private, allocatable :: face_normals(:,:)   ! 面の法線。
        real(WP_), private, allocatable :: face_centers_(:,:)   ! 面の中心。   
        !real(WP_), private, allocatable :: face_cell_centers(:,:)
        !real(WP_), private, allocatable :: face_cell_distances(:)      
        real(WP_), public, allocatable :: face_velocities(:,:)    
        
        real(WP_), private :: gamma_ = 0
    
        
        type(backward_difference_t), private :: bdfs_
    contains            
    
        procedure, public :: init
        procedure, public :: update
        generic :: apply_boundary_condition => apply_boundary_condition_r1_, apply_boundary_condition_r2_, apply_boundary_condition_r3_
        
        procedure, private :: apply_boundary_condition_r1_
        procedure, private :: apply_boundary_condition_r2_
        procedure, private :: apply_boundary_condition_r3_
        
        
        
        !============================== I/O overload =================================
        procedure, private :: unformatted_write_
        procedure, private :: unformatted_read_
        generic :: write(unformatted) => unformatted_write_
        generic :: read(unformatted) => unformatted_read_
        !============================= End I/O overload ===============================
    end type  

このクラスは ``grid_t`` クラスのメンバを別途保持している. メンバ関数 ``apply_boundary_condition`` で
境界条件クラスの ``evaluate`` を呼び出す.  処理内容は, あるゾーンの境界面1つ1つに対して境界条件クラスの ``evaluate`` 関数を
呼び出すというものである.

.. 圧力Poisson方程式
.. ------------------

.. 圧力Poisson方程式にたいする境界条件は若干異なる. 

**References**

.. [1] 日野幹雄, 流体力学, 朝倉書店, p.119