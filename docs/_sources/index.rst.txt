.. afdet_user_guide documentation master file, created by
   sphinx-quickstart on Thu Mar 10 22:51:07 2022.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to afdet solver's user guide!
============================================

afdet solver は 非構造移動格子有限体積法のためのフレームワークです.

本ソルバは流体計算の実行プログラムと, それを構成するためのライブラリを
ユーザに提供しています. ユーザはあらかじめ用意された機能拡張用クラスを用いて計算の前処理や毎ループ後の後処理, 
境界条件やデータ書き出し処理などを独自にカスタマイズすることが出来るため, 過去の計算プログラムと比べて比較的柔軟かつ
安全に独自の機能を盛り込むことが出来る. また拡張クラスと補助サブルーチンの作成に当たって, 
ユーザは ``lib\common, lib\common2`` 内のライブラリ群のサポートを受けることが出来るため, 
これらのライブラリを上手に組み合わせて効率よく開発を進めることが出来るのも特徴のひとつである.

このユーザガイドは, ソルバの基本的な使い方や用語の解説を行い, 
ユーザの開発を支援することを目的としています. 

このユーザガイドの著者は T.I.です. 協力者を募集しています．

.. note::
   本ユーザガイドは非圧縮性流体計算用フレームワーク ``afdetFractional`` の内容を基にしています．
   圧縮性流体用のフレームワークとは若干違うかもしれません．またバージョン ``1.0.8`` ~ ``1.2.0`` に基づいた記述なので，
   アップデートによってはAPIが変更される可能性もあります. ここにかかれているプログラムの実装内容は参考程度にとどめておいてください.
   フレームワークの作成者は本ガイドの著者とは異なるので，プログラムの詳しい内容については作成者本人と連絡を取ってください．

.. toctree:: 
   :maxdepth: 2
   :caption: ソルバのインストールと計算の手順:
   :numbered:

   howtoinstall
   howtocalc

.. toctree:: 
   :maxdepth: 2
   :caption: ソルバの基本事項:
   :numbered:

   /main/fvm_mesh_data.rst
   /main/geometry.rst
   /main/boundary_condition.rst
   /main/gradient_scheme.rst
   /main/incomp_simulator.rst

.. toctree:: 
   :maxdepth: 2
   :caption: その他のライブラリ:
   :numbered:

   /libs/quaternion.rst
   /libs/rigidbody.rst
   /libs/backward_diff.rst
   /libs/springmesh.rst

.. toctree:: 
   :maxdepth: 2
   :caption: ソルバのカスタマイズ:
   :numbered:

   /custom/setup_boundary_zone.rst
   /custom/makeBC.rst
   /custom/plugin_case.rst
   /custom/use_intel_mkl.rst

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
