KLiH3のconvex hull計算
==========================================

水素化物ペロブスカイトの一種である :math:`\ce{KLiH_3}` という物質が、在る圧力下において安定であるかどうかについて、そのconvex hullを計算することで判定したい。

**やること**

1. 計算を行う圧力の配列を用意する
    - できればフレキシブルに配列を用意したい
    - 用意した配列の圧力で :code:`*.vc-relax.in` ファイルの :code:`press` の値を書き換えないといけない
    - nodeのfor構文で、圧力値を引数としてbashスクリプトを回して、その各物質のエンタルピーのリストを受け取る。
    - そのリストからplotlyに渡すデータを作成する。

2. 各圧力において、 :math:`\ce{H_2}` 、 :math:`\ce{KH}` 、 :math:`\ce{LiH}`、 :math:`\ce{KLiH_3}` のvc-relax計算をする
    - bashから、渡された圧力値でpressを書き換え、その上で圧力書き換え済みの各物質の :code:`*.vc-relax.in` ファイルを実行する。

#. :code:`Final enthalpy` の数値を :code:`*.vc-relax.out` から抽出する
    - 以上の1~3を見るに、圧力の書き換え処理とその圧力での各物質のvc-relax計算とその計算結果のエンタルピーの抽出まではbashスクリプトを組んで実行するべき。

#. KLiH3とその相分離後の物質とのエンタルピーをそれぞれ計算する
#. この計算を各圧力に対して繰り返す
#. データのプロットを行う

**注意すべきこと**

.. note::

	計算資源は多くはないので、できるだけ少ない計算量にしたい。したがって計算する圧力は最初はログスケールで増やしていき、そのあとで、もし必要であればエッセンシャルな圧力範囲を抽出して計算する。

.. note::

	また、バンド計算は必要がない。なぜなら :code:`Final enthalpy` は :code:`*.vc-relax.out` の中に入っているから。

成果物出力までの技術選定
-----------------------------------
- electron
    * メインプロセス
    * レンダープロセスがリッスンしているフロントエンド入力からイベント発火か、もしくはメインプロセスとの通信で、nodeを経由してbashスクリプトを実行する（あるいはnodeから直接QEを動かすか？)でのコード実行を担う。（セキュリティ的にサニタイズする必要がすごくある）
- node
    * bashスクリプトに引数を渡して実行しないといけない。
    * bashスクリプトの結果を受け取ってパースしてjsの配列を作成しないといけない
- React
    * 圧力配列を入力として受け取るインターフェースが必要
    * ??入力に応じた、nodeによるbashスクリプト呼び出しのトリガーはどうする？
- bash
    * 与えられた圧力に対して４種の物質のエンタルピーを計算するQuantumEspressoの振る舞いを制御する。
    * 計算結果の４種のoutputファイルからエンタルピーを抽出し、json形式にまとめる。

    .. code-block:: js

        {
            "圧力": {
                "1d01": {
                    "KH": "-133.3",
                    "LiH": "-33.3",
                    "H2": "-13.3",
                    "KLiH3": "-144.4"
                },
                "1d02": {
                    // 原子種は同様
                },
                "5d02": {
                    // 同様
                },
                "1d03": {
                    // 同様
                },
                // 以下同様に圧力が続く
            },
        }

計算する圧力の選定
-----------------------------------


水素について
----------------------------------

高圧下における水素の結晶
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
木星中心における水素の金属化が、木星の地磁気の強さの原因なのでは？という動機から高圧水素の圧縮研究が進む。

400万気圧くらいで金属化する実験結果（結構確度が高い）

高圧下のおける水素の結晶構造は実験的には分かっていない。

1Tpa~原子状のダイヤモンド構造が最安定？

- KMgH3のMgをLiで少し置き換える
