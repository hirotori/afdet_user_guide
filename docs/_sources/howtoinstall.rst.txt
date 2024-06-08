========================
インストールに必要なもの
========================

PC
---------

言わずもがな. ラップトップでは厳しい. デスクトップマシン推奨.
OSについては, 基本的にはUNIX系を使用する. 以下の環境での動作が確認されている.

.. csv-table:: 
    :header: OS, version
    :widths: 5, 5

    Ubuntu, 20.0
    CentOS, 7.0

コンパイラ
----------

あらかじめ Fortranコンパイラを用意する必要がある.

デフォルトは ``intel fortran`` である. 


後述の ``Makefile`` を書き換えれば変更は可能.

.. warning:: 
    コンパイラの変更は可能だが, ソルバ内のサブルーチンの幾つかが ``intel Fortran`` に依存しているため, 
    他のコンパイラを使いたい場合はそのあたりの修正を念頭に置くこと.


ビルドツール
------------

本ソルバはコンパイルに ``Makefile`` を使用する. 

.. _インストール手順:

================
インストール手順
================

ソルバ一式を入手する. 入手方法は, 今のところ開発者に連絡してzipファイルを貰うしかない.

適当な場所に入手したファイルを解凍する. ディレクトリ構成は以下の通りとなっている.

.. code-block::
    
    afdet_solver-###(###はバージョン番号.)/
        ├ DOC(ver1.0.8以前. それ以降含まれず)/
        ├ sample/
        |   ├ case_poiseuille/
        |   └ project_piston
        └ solver_project/
            ├ lib/
            |   ├ common/
            |   ├ common2/
            |   ├ mod/
            |   ├ obj/
            |   └ Makefile
            ├ obj/
            ├ bin/
            ├ src/
            └ Makefile

``solver_project`` ディレクトリに移動し, 以下のコマンドをたたく.

.. code-block:: 

    ~$ make

このコマンドにより, ``/solver_project/lib/common, common2`` 内のライブラリ, および ``solver_project/src`` 内の
メインプログラムのビルドを開始する.

.. note:: 

    ``make:  'all' に対して行うべき事はありません`` とでた場合, 既にビルド済みである可能性が高い.
    しかし他のPC環境でビルドしたバイナリは使えないことが多いので, ``make clean`` として生成物を全て消去する(ソースファイルは消えないので大丈夫.)

.. note:: 
    当然だが ビルドツール ``Makefile`` が必要. 

.. figure:: /images/install_process1.png

    コマンドを実行するディレクトリと,端末画面の例

ビルドが終了すれば,以下のような画面が出てくる(ただしバージョンによって細部は異なる.)

.. figure:: /images/install_process2.png

    make終了時の画面例

.. note:: 

    もしビルドが途中で止まる場合は開発者に連絡してください.


ビルドが正常に終了した後, 以下のコマンドによりシステムへのインストールを開始する.

.. code-block:: 

    ~$ sudo make install

インストール先は ``/usr/local/lib`` と ``/usr/local/bin`` である.
成功すると以下のような画面が表示される.

.. figure:: /images/install_process3.png

    success と表示されればインストール完了.

正常にインストール出来ているか確認するために, ソルバのコマンドを実行する(非圧縮性ソルバの場合).

.. code-block:: 

    ~$ afdetFractional

上手くいけば

.. code-block:: 

    ~$ afdetFractional
      KIT Flow solver afdetFractional ver.1.0.8

    Usage:
      afdetFractional <case directory>

と表示されるはず.