=============================
Intel MKL を利用する方法
=============================

概要
================

機能を追加する上で, 擬似逆行列や特異値分解が必要になるケースが有り得る.
Intel Math Kernel Library (MKL) は intel製アーキテクチャ向けに最適化された
行列計算やFFT, マトリクスソルバ等を含む高機能ライブラリ群である. 
自作するよりも遙かに高性能であり, かつ信頼性の高いライブラリであるためバグの混入を
避けることが出来る点が魅力的であるが, 利用のための手順がややこしいのでここにまとめる.

手順
================

Intel MKLのインストール
------------------------

Intel OneAPIのHPC Toolkitに含まれる. とくにインストールオプションを選択していない限りはインストール出来ている.
場所はLinuxであれば ``\opt\intel\mkl`` で存在を確認することが出来る.

プログラムの作成
------------------------

BLASを使いたければ ``use blas95`` , LAPACKなら ``use lapack95`` とする.
後ろの ``95`` とはFortran95 インタフェースのことであり, オリジナルよりも引数の個数が軽減され使いやすくなっている.

例として, 実特異値分解により連立方程式 :math:`{\bf A}{\bf x} = {\bf b}`  を行う場合を考える. 次のように書く.

.. code-block:: fortran

    subroutine solve_by_svd3(A, b, x, info)
        use lapack95
        use blas95
        implicit none
        real(wp_),intent(inout) :: A(3,3)
            !!3×3行列
        real(wp_),intent(in) :: b(3)
            !!定数ベクトル
        real(wp_),intent(inout) :: x(3)
            !!解ベクトル
        integer,intent(out) :: info

        real(wp_) :: U(3,3), VT(3,3), S(3), WW(2)
        real(wp_) tmp_(3)

        call gesvd(A, S, U, VT, WW, info = info)
        call gemv(U, b, tmp_, trans="T")
        tmp_(:) = tmp_(:)/S(:)
        call gemv(VT, tmp_, x, trans="T")

    end subroutine

``gesvd`` で実特異値分解サブルーチンを呼び出している. ``gesvd`` は特異値分解を行うサブルーチンの総称名であり, 
引数の型に合わせて適切なサブルーチンが呼び出される.

``gemv`` はベクトルと行列の積を計算するblas level2 ライブラリである. これも総称名.

MakeFileへの記述
----------------------
プラグイン ``afdetUCC.so`` の作成のためにmakefileを記述する. このとき, インストールされたライブラリを適切に
リンクさせる必要がある.

まずMKLライブラリを使用するために, コンパイルオプションは ``-qmkl -shared-intel`` とする. 
そもそもこれがないとコンパイル済みの ``libafdet.a`` に上手くリンク出来ない.

このままコンパイルすることもおそらく可能であるが, メインプログラム実行時に ``symbol look up error ::`` なるものが出てランタイムエラーとなると思われる.
``-qmkl`` だけではアクセス出来ていないため?

そこで, `Intel® oneAPI Math Kernel Library Link Line Advisor <https://www.intel.com/content/www/us/en/developer/tools/oneapi/onemkl-link-line-advisor.html#gs.6e5df2>`_ を利用する.
上から順番に項目を指定して, 最後に出力されるリンカ行とオプション行をコピーし, それぞれ ``LDFLAGS, INCLUDE`` に追加する.

.. note:: 

    Select interface layer: は ``Fortran API with 32-bit integer`` を選択する必要がある. 本ソルバは 32bitの整数を利用しているためである.

