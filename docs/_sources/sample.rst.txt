=====
TEST
=====

これはテスト. 

https://planset-study-sphinx.readthedocs.io/ja/latest/04.html#id10 を参考に. 

サブセクション
=================

**強調** する. 強調構文と普通の文の間には半角スペース. 

``intro.rst`` の内容. 

数式
^^^^^

http://sphinx.shibu.jp/ext/math.html を参考にする. 

インライン数式 :math:`a = b + c` 

あるいは行まるごと. 

.. math::
    :label: integral_x

    x = \int_{\Omega_{s}} \nabla \cdot {\bf u} d\Omega

式番号はラベル付けしてから, おなじドキュメント内で :eq:`integral_x` のように参照. 

サブサブセクション
==================

コードブロック. これも1行開けて書く. 

.. code-block:: fortran

    module sample
    use other_module, only : hogehoge
    implicit none
    type sample_t
        integer ii
        real,allocatable :: rx(:)
        contains
        procedure,private :: hoge => huga
    end type

    type(sample_t),private :: myclass
    end module

    program main
      implicit none
      integer i
      real x
      
      x = 0.0
      do i = 1, 10
        x = x + 1
      end do

    end program