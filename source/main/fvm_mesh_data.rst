=================
メッシュデータ
=================

本ソルバは非構造格子を取り扱う. 構造格子と異なり, 非構造格子では格子点やセルの番号付けに
規則性がない. そのため, ある格子の **隣接関係** (connectivity)は解析を行う前に配列として
用意しなければならない. 

ここでは本ソルバで用いられるメッシュとそのデータを取り扱うクラス ``grid_t`` について説明する. 

セル(cell)
============
物理量を定義する図形を **セル** (cell)あるいは **要素** (element)と呼ぶ.
本ソルバは3次元流れ場のためのプログラムで, セル中心型の有限体積法を採用しているので, 
物理量は3次元立体の重心に定義され, また **検査体積** (control volume)は領域内の各3次元立体にとられる.

セルの形状
----------

.. figure:: /images/cell_types.png

    利用可能なセル形状.

本ソルバでは以下の4種類のセルが使用出来る. 

* 四面体(tetrahedron)
* プリズム(wedge)
* ピラミッド(pyramid)
* 六面体(hexahedron)

セルは複数枚の **面** (face)で構成される. 面は複数の **節点** (node)で構成される. 
各セルが持つ節点および面の個数は下表の通りである. 

+-----------+-------+-------+-------------------------+
|cell type  | node  | face  | face type               |
+===========+=======+=======+=========================+
|tetrahedron|   4   |   4   | triangle                |
+-----------+-------+-------+-------------------------+
|wedge      |   6   |   5   | triangle & quadlilateral|
+-----------+-------+-------+-------------------------+
|pyramid    |   5   |   5   | triangle & quadlilateral|
+-----------+-------+-------+-------------------------+
|hexahedron |   8   |   6   | quadlilateral           |
+-----------+-------+-------+-------------------------+

下図に2次元メッシュにおけるセル,面,節点の関係を示す.

.. figure:: /images/cell_face_node.png

    セル,面,節点の関係


セルにはいくつかの種類がある.  計算領域内に存在するものを **内部セル** (inner cell) あるいは単にセルとよぶ ( :numref:`fig_bnd_and_inner_cells` 参照). 
それとは異なり, 計算領域の境界より外側に一層だけ, 仮想的にセルを割り当てており, 
これを **ゴーストセル** あるいは **仮想セル** (ghost cell, dummy cell) と呼ぶ.
内部セルとは異なり可視化時には現れず, 計算上の都合で割り当てられるためにこう呼ぶ. 仮想セルを設けることで, 境界面に隣接するセルを内部セルと同じ離散式で取り扱うことが出来る.
詳しくは :doc:`/main/boundary_condition` を参照.

.. figure:: /images/bnd_and_inner_cells.png
    :name: fig_bnd_and_inner_cells

    境界セル(図中灰色)と内部セル(白).

なおセルには番号が付けられている. ``1 ~ cell_count`` が計算領域内部に存在するセル(inner cell), 
``cell_count + 1 ~ cell_count2`` が計算境界の外側に設けられた仮想セル(ghost cell) である.
``cell_count`` は内部セルの個数を表しており, ``grid_t`` クラスのメソッドから取得する.


内部セル(inner cell)
---------------------
:numref:`fig_bnd_and_inner_cells` の白いセルが該当する.

計算領域中のセルの内, 境界に接していないセルを任意に1つとると, そのセルは自身の持つ面それぞれに
1つ隣接するほかのセルを持つ. したがって隣接するセルの個数は面の個数に一致する. 

例外は1つの四角形面を三角形に分割する場合である. このとき, 2つの面が同一のセルと
隣接するので, 個数は一致しない. 

.. note::

    現バージョンでは三角形分割はされていない. 
    どうしても三角形分割が必要な場合は, ``grid.f90`` 内のマクロ変数 ``TRIANGULATE_FACE`` を
    1に変更し, 再度コンパイルを実施する.



境界セル(boundary cell)
---------------------------
:numref:`fig_bnd_and_inner_cells` の灰色のセルが該当する.

内部セルに含まれるが, セルのある面が計算領域の **境界** (boundary,boundary surface)と接しているものは特別に **境界セル** (boundary cell)と呼ぶ.
境界セルの面のうち少なくともひとつは
ほかのセルと隣接しない. このような面を **境界面** (boundary face)とよぶ. 
ただし, 境界条件の設定を簡単にするため本ソルバは境界面を挟んだ外側に仮想的なセルを一層設けている. 
そのため, 境界面も実際には隣接するセルを持つ. このような仮想的なセルを仮想セル(ghost cell)と呼ぶ. 

ゴーストセル(ghost cell)または仮想セル(dummy cell)
---------------------------------------------------
境界の一層外側に設けられたセルのことで, 境界条件を簡単に実装するために用意されている.
仮想セルにも物理量は定義されるが, 境界面の値と,それに対応する境界セルの値から境界条件に応じて決定される.
詳細は :doc:`/main/boundary_condition` を参照. 
なお仮想セルは, :numref:`fig_boundary_and_ghost_cells` のように, 境界セルが境界面を挟んで対象となる位置に配置される.

.. figure:: /images/boundary_and_ghost_cells.png
    :name: fig_boundary_and_ghost_cells

    ゴーストセルは境界面を挟んで鏡面対象となる位置に配置される.

面(face)
=================

セルを構成する多角形. セルにねじれが生じる場合は平面図形ではないが, そのようなことは稀なので
平面図形と見なして処理される. 

セル毎に面を数えると重複するので, 重複の無いように1つずつ別個に番号付けされている. 
境界面以外の面は全て **内部面** (inner face) あるいは単に面 と呼ぶ. 面は方向も含めて定義されている.
これはつまり, セルから面を見たとき, 面の向きは必ずしもセルの外向きとは限らないことを意味する.

.. note:: 面の向きについて

    面の向きは, それを挟む2つのセル番号 c1, c2について, 
    番号が小さい方から大きい方へ (c1 < c2) 向くように設定されている.
    面の向きの計算は :doc:`/main/geometry` を参照.


内部面に対して, 計算領域の境界表面(boundary surface)に接する面を **境界面** (boundary face)と呼ぶ.


節点(node,point)
=================

文字通りの座標点. それぞれが座標成分を持つ. 節点は面を構成する. 
本ソルバはセル中心型なので,物理量は節点では無くセルの重心に定義される ( :ref:`重心の計算` 参照).


本プログラムのデータ構造
=============================
本ソルバは非構造格子(unstructured mesh)を採用しているため, セル,面,節点番号は構造格子と異なり
不規則に並んでいる. そのため, 接続関係などのテーブルを別途配列として保持する必要がある.
接続関係の構築や定義は, おおよそ以下のソースファイルを辿れば理解出来る.

* ``unstructured_mesh.f90``
* ``grid.f90``

ここではデータ構造の構築方法には触れず, 構築されたデータをどのように扱うかについて焦点をあてる.

データを扱うクラス
------------------------
非構造格子の接続関係,幾何量などのデータは ``grid_t`` クラスが受け持つ. データ構造に関するものとして,
例を挙げると

* ``cell_offsets``
* ``cell_faces``
* ``face2cell``

などがある. これらは ``grid_t`` クラスのメンバである(ただしカプセル化はされていない.).

データのアクセス方法
---------------------

セルから面へ
^^^^^^^^^^^^^^
``cell_offsets`` と ``cell_faces`` を用いてアクセスする. 
``cell_offsets`` は ``cell_faces`` の開始オフセットである. つまり, 
あるセルの面へアクセスするためには, 配列 ``cell_faces`` の ``cell_offsets(cell)`` 番目からの値を見れば良い.

.. figure:: /images/cell2face.png

    セルから面へアクセスする. セルから見て法線が内向きの場合, ``cell_faces`` は負値をとる.
    図の右下がデータ構造のイメージである. 配列 ``cell_faces`` を ``cell_offsets(cell)`` 番目(赤字)
    から ``cell_offsets(cell+1)-1`` 番目までアクセスして(図中青枠)面番号を取得する.


アクセスの例:

.. code-block:: Fortran

    do cell = 1, cell_count
    do offset = cell_offsets(cell), cell_offsets(cell+1)-1
        face   = abs(cell_faces(offset))

        !いろいろやる

    end do
    end do

セルから隣接セルへ
^^^^^^^^^^^^^^^^^^^
``cell_faces`` はセルから見て面の法線が外向きなら正値, 逆なら負値をとる. それを利用して隣接セル番号を取得出来る. 

以下に一例を示す. 

.. code-block:: Fortran

    do cell = 1, cell_count
    do offset = cell_offsets(cell), cell_offsets(cell+1)-1
        face   = cell_faces(offset)

        !cell_faces < 0 なら face2cell(1,face), 
        !cell_faces > 0 なら face2cell(2,face)
        nbcell = face2cell((3 + sign(1, face))/2, abs(face))

        !いろいろやる

    end do
    end do


あるいは ``grid_t`` のメソッド ``cell_next_cell`` を利用する. これは ``elemental`` 属性なので, 
引数 ``side`` に配列( ``[i, i = cell_offsets(cell), cell_offsets(cell+1)-1]`` )を渡せばまとめて隣接セルを
取得出来る. 

.. code-block:: Fortran

    pure elemental integer function cell_next_cell(this, cell, side)
        class(grid_t), intent(in) :: this
        integer, intent(in) :: cell, side
        integer face
        face = face_next_cell(this, cell, side)        
        cell_next_cell = this%face2cell((3 + sign(1,face))/2, abs(face))
    end function

.. _面からセルへ :

面からセルへ
^^^^^^^^^^^^^^

``face2cell(1:2,face)`` をつかう. 境界面に対しては ``face2cell(2,face)`` がゴーストセルを指す. 
イメージを以下の図に示す.

.. figure:: /images/face_cell_connectivity.png

    面からセルへのアクセスの概念図.

.. note:: 

    ``face2cell(1,face) < face2cell(2,face)`` となるように定められている. ただし, あくまで面を挟む2つのセルに対する
    アドレスであって, その面がどちらのセルに属するかについてはわからない. 
    面の向きについては, セルから面へのアクセスを用いる.

面から節点へ
^^^^^^^^^^^^^^^

``face2vertex(nv,face)`` を用いる. 第1引数は節点のローカル番号で, 有効範囲は ``nv = 1, face_vert_counts(face)`` . 

境界面
^^^^^^^^^^^^^^

境界面だけにアクセスするための配列が用意されている. 境界条件を適用させる領域を境界ゾーン(boundary zone)と呼ぶ 
(:numref:`fig_zone_and_boundary_faces`) ことにすると, 
各ゾーン番号に対する境界面は以下のようにアクセス出来る. 

.. code-block:: Fortran

    do zone = 1, zone_count            
        do offset = boundary_offsets(zone), boundary_offsets(zone+1) - 1
            face = boundary_faces(offset)
            !いろいろやる
        end do
    end do

.. figure:: /images/zone_and_boundary_faces.png
    :name: fig_zone_and_boundary_faces

    円柱周りの流れにおける境界ゾーンの定義の例.

ここで ``zone`` は境界ゾーンの番号である. 番号の指定については X を参照.
``boundary_offsets`` は ``cell_offsets`` と同様に,
指定した ``zone`` の境界面 ``boundary_faces`` にアクセスするための
オフセット配列である. 例を以下に示す.

.. code-block:: 

    boundary_faces = [1,2,23,...,678,...]
                      ^           ^
                      |           |
                      1          100
    boundary_offsets = [1, 101, 302, ...]

``boundary_offsets`` は各ゾーンの ``boundary_faces`` の開始インデックスである.
``zone = 1`` の場合は, ``boundary_faces(1:101-1)`` がそのゾーンに属する境界面となる.


節点からセルへ
^^^^^^^^^^^^^^^^
基本的にはセルから面へのアクセスと同じだが, 範囲は ``vertex_cell_bounds(2,vert)`` であることに注意. 

.. code-block:: Fortran

    do vert = 1, vert_count
        do offset = vertex_cell_bounds(1,vert), vertex_cell_bounds(2,vert)
            cell = vertex_cells(offset)
            !いろいろやる
        end do
    end do

その他
=================

慣れないうちは ``grid_t`` のメンバ関数を利用すると良い. 