====================================
境界ゾーンのファイル指定
====================================

境界ゾーンの指定方法は

1. ユーザ定義クラスの ``on_define_zone`` から指定
2. 表面メッシュ(.obj)から指定

の2通りがある. 


ユーザ定義クラスの ``on_define_zone`` から指定
----------------------------------------------
汎用ケースクラス ``case_common_t`` を継承し，メソッド ``on_define_zone`` をオーバーライドして，メッシュ形状に合うように境界条件を指定する．
仕様については :ref:`pre_process` の ``on_define_zone`` を参照．

例えば，直方体の計算領域を用意して平行平板間流れを解こうとする場合を考える．流れ方向を :math:`x` , 壁に垂直な方向を :math:`z` とすると, 

.. code-block:: Fortran

    subroutine on_define_zone(this, grid_id, face_center, face_normal, zone_id)    
        class(case_pois_t), intent(in) :: this
        integer, intent(in) :: grid_id
        real(WP_), intent(in) :: face_center(3)
        real(WP_), intent(in) :: face_normal(3)
        integer, intent(out) :: zone_id
    
        real(WP_), parameter :: Eps = 0.0001_WP_
        
        if ( face_normal(1) <= -1.0_wp_ ) then
            zone_id = ZoneInlet_
        else if ( face_normal(1) >= 1.0_wp_ - Eps) then
            zone_id = ZoneOutlet_
        else if ( face_normal(2) <= -1.0_wp_ ) then
            zone_id = ZoneLeft_
        else if ( face_normal(2) >= 1.0_wp_ - Eps) then
            zone_id = ZoneRight_
        else if ( face_normal(3) <= -1.0_wp_ ) then
            zone_id = ZoneWall_
        else if ( face_normal(3) >= 1.0_wp_ - Eps) then
            zone_id = ZoneWall_
        end if
    end subroutine


ここで ``zone_id`` に与える数字は ``ZoneInlet_`` などのようにモジュールで定数として定義しておく．

.. code-block::  Fortran

    module case_poiseuille_m
        ! ...

        integer,parameter :: ZoneInlet_ = 1
        integer,parameter :: ZoneOutlet_ = 2
        integer,parameter :: ZoneLeft_ = 3
        integer,parameter :: ZoneRight_ = 4
        integer,parameter :: ZoneWall_ = 5


表面メッシュから指定
----------------------------
.objの仕様は例えば `Wikipedia <https://ja.wikipedia.org/wiki/Wavefront_.obj%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB>`_ 
などを参照して貰うとして, 本ソルバで用いる形式は以下の通りである.

.. code:: 

    # ジオメトリの頂点
    v 0.0 0.0 0.0
    v 0.0 0.0 1.0
    :

    # 法線(任意). 単位ベクトルでなくても良い.
    vn 0.707 0.000 0.707
    vn ...

    # mtlは未対応.
    # グループ. 次にそのグループのポリゴン面要素
    # 1つのグループが1つの境界に対応する.
    g group1
    f 1 2 3
    f 3/1 4/2 5/3
    f 6/4/1 3/5/3 7/6/5
    f ...

**境界1つあたり1つのファイルでは無く**, 1つの格子あたり1つの.objファイルを指定する.
1つの.objファイルに全ての境界面の情報を書き込む.
頂点座標は1つのファイルに共通で, 面要素だけをグループ毎に指定することで各境界を区別する.

.. note:: 

    beta版での情報. 今後変わるかも知れない.

ファイルの指定
-----------------------------
``case_setting.txt`` に指定する.