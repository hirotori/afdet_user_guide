==============================
境界条件の作成例
==============================
:doc:`/main/boundary_condition` で示したとおり, 境界条件は境界条件クラスで実装されている.
これらのクラスを任意に拡張することで, ユーザ独自の境界条件を定義することが可能である.
以下では, ユーザ定義の境界条件クラスの例を挙げ, 具体的な拡張の仕方を提示する.

放物線分布の流入条件
==============================
パイプ内流れやチャネル内流れを計算する場合, 流入条件を一様流としてしまうと, 
速度分布が完全に発達するまでの距離(助走区間)だけ余分に計算領域が必要である.
計算格子数を節約するため, 一様流入ではなく完全に発達した層流ポアズイユ流を与えることを考える.

クラスの定義
------------------------
流れ方向を :math:`x` 軸, 残りを :math:`y,z` 軸とする. 半径 :math:`R` の円管を流れる層流を
考えるとしよう. 無次元化のため代表速度を :math:`U_{\infty}` , 長さを :math:`L_{\infty}` と置くと, 
無次元速度分布は次式で表される.

.. math:: 

    u = \frac{\alpha}{4}Re\left\{ \left(\frac{R}{L_{\infty}}\right)^{2} - r^{2} \right\}

ただし :math:`r = \sqrt{y^{2} + z^{2}}` である. :math:`Re` はレイノルズ数で, :math:`\alpha` は
無次元圧力勾配である.  

この速度分布を境界面に与える. 親クラスとして ``fixed_value_patch_t`` を選び, 
``bc_laminar_pipe_t`` と名付ける. 定義は以下の通りとした.

.. code:: fortran

    type, extends(fixed_value_patch_t) :: bc_laminar_pipe_t
        !!円管内ポアズイユ流の流速分布を与える.
        real(wp_) :: cross_avg_vel(3) = [0,0,0]
            !!断面平均流速. 
        real(wp_) :: origin(3) = [0,0,0]
            !!面の中心. 外部で更新可能
        real(wp_) :: Re_ = 0.0_wp_
            !!レイノルズ数
        real(wp_) :: dpdx_ = 0.0_wp_
            !!圧力勾配
        real(wp_) :: pipe_radius_ = 0.0_wp_
            !!円管の半径(無次元)
        CONTAINS
        PROCEDURE :: evaluate => fully_developed_pipe_laminar_flow
    end type

メンバ変数として ``cross_avg_vel`` と ``origin`` を用意している. 
これは任意の向きにも対応できるようにするための配慮である. ただし ``cross_avg_vel`` は大きさが1となる必要がある.
ほか, ``Re_`` , ``dpdx_`` も必要である. これらの値の初期化はコンストラクタを利用する.

このクラスはまだ型束縛手続き ``evaluate`` が未定義である. 上では ``fully_developed_pipe_laminar_flow`` という関数を定義し,
``evaluate`` と結合している.

.. note:: 

    名前は ``evaluate`` でも良いが, 1つのモジュール内に境界条件の複数の子クラスを書く場合は名前の衝突が起こるため,
    別名で定義する.

以下のように定義した.

.. code:: fortran

    PURE SUBROUTINE fully_developed_pipe_laminar_flow(this, q, face_velocity, face_center, face_normal, distance, prim_count, ghost_q)
        CLASS(bc_laminar_pipe_t), INTENT(INOUT) :: this
        REAL(WP_), DIMENSION(:), INTENT(IN) :: q
        REAL(WP_), DIMENSION(3), INTENT(IN) :: face_velocity
        REAL(WP_), DIMENSION(3), INTENT(IN) :: face_center
        REAL(WP_), DIMENSION(3), INTENT(IN) :: face_normal
        REAL(WP_), DIMENSION(2), INTENT(IN) :: distance
        INTEGER, INTENT(IN) :: prim_count
        REAL(WP_), DIMENSION(:), INTENT(OUT) :: ghost_q

        real(wp_) vel_(3), radius_

        radius_ = norm2(face_center - this%origin)

        vel_(:) = 0.25_wp_*this%cross_avg_vel(:)*this%Re_*this%dpdx_*(this%pipe_radius_*this%pipe_radius_ - radius_*radius_)

        ghost_q(:) = calc_ghost_value(q, vel_, distance(1), distance(2))

    END SUBROUTINE


クラスの呼び出し
------------------------
作成したクラスを実際の計算で使用するため, ケースクラス ``case_common_t`` を拡張する必要がある.
(ここでは ``case_pois_t`` としている)
:doc:`/custom/plugin_case` で述べた ``create_boundary_condition`` を利用する.

だいたい以下のようになる.

.. code:: fortran

    function create_boundary_condition(this, bc_type, values) result(bc)
        class(case_pois_t), intent(inout) :: this
        integer, intent(in) :: bc_type
        real(WP_), intent(in) :: values(:)
        class(patch_field_t), pointer :: bc

        type(bc_laminar_pipe_t) :: bc_laminar_pipe_
        integer, parameter :: YourBC = 8
            !!独自の境界条件番号を定義する.
        real(wp_), parameter :: PipeRadius = 0.5_wp_
            !!無次元円管半径.(R/L∞) 
        real(wp_), parameter :: DPDX_ = 1.0_wp_
            !!無次元圧力勾配.

        if (bc_type == YourBC) then
            !デフォルトのコンストラクタを呼び出し, クラスを初期化する.
            bc_laminar_pipe_ = bc_laminar_pipe_t(values(1:3), [0,0,0], this%Re_, DPDX_, PipeRadius)
            !返値の境界条件オブジェクトに割り付ける.
            allocate(bc, source = bc_laminar_pipe_)
        else
            bc => create_bc_default(bc_type, values)    ! Don't remove.
        end if
    end function

仮引数のケースクラスにはレイノルズ数がメンバとして与えられていないので, 
ここでは苦肉の策としてユーザ拡張クラスで独自に定義し(``this%Re_``), ``on_loaded`` 関数で更新することにした.

.. code:: fortran

    subroutine on_loaded(this, grid, fluid, is_restart)
        class(case_pois_t), intent(inout) :: this
        type(grid_t), intent(inout) :: grid
        type(fluid_t), intent(inout) :: fluid
        logical, intent(in) :: is_restart

        !(中略) ...

        this%Re_ = fluid%reynolds_number

        !(中略) ...

    end

ハードコーディング(直に与える)でも良いが, レイノルズ数は別の箇所で定義されるので間違える可能性もあることを考慮している.
