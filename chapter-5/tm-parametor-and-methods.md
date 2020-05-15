---
description: TM Parametor and Methods
---

# TM のパラメーター・メソッド

**`TemporalMemory()`** は時間プーラーを実装したクラスであり、ミニカラム内セルの関係を扱います。

この関数の主要なインタフェースはSPと同様 `compute()` ****メソッドであり、入力ベクトルを受け取り、セル間の学習と予測をおこないます。

{% code title="python3" %}
```python
tm = TemporalMemory(...)
for line in file:
  inputVector = numpy.array(line)
  tm.compute(inputVector, learn=True)
  ...
```
{% endcode %}

内部情報はメソッドによりトレースすることができます。時系列予測をおこなう際は、**`tm.getActiveCells()`** でトレースしたアクティブセルをもとに実際の入力と比較し、学習器を訓練します。

**パラメータ**

* **`inputDimensions`** ：入力ベクトルの次元を表す整数のリスト。フォーマットは \[height, width, depth, ....\] で，各値は次元のサイズを表します。
* **`columnDimensions`**：リージョン内のミニカラムの次元を表す整数のリスト。書式は \[height, width, depth, ....\] で，各値は次元のサイズを表します。
* **`potentialRadius`**：各ミニカラムが潜在的に接続可能な入力の範囲を決定します。このパラメータは、正方形（またはハイパースクエア）領域を定義します。
* **`numActiveColumnsPerInhArea`**：ミニカラムの潜在的な半径内で、接続可能な入力のパーセンテージ。1に設定すると、ミニカラムは潜在的な半径内のすべての入力に接続されます。
* **`globalInhibition`**：true の場合、抑制フェーズの間、勝利したミニカラムは、リージョン全体の中で最もアクティブなミニカラムとして選択されます。
* **`localAreaDensity`** ：局所的な阻害領域内のアクティブ・カラム

  の密度をあらわします。

* **`numActiveColumnsPerInhArea`**：アクティブなカラムの分散度を制御する別の方法です。
* **`stimulusThreshold`** ：ミニカラムをオンビットにするためにアクティブにしなければならないシナプスの最小数を指定した数値です。これは、ノイズの多い入力がミニカラムを活性化させないようにします。
* **`synPermInactiveDec`**：シナプスの永続性を、学習ステップごとに減少させる量。
* **`synPermActiveInc`**：シナプスの永続性を、学習ステップごとに増加させる量。
* **`synPermConnected`**：デフォルトの接続しきい値。
* **`minPctOverlapDutyCycle`**：0 から 1.0 の間の数値で、列が少なくとも stimulusThreshold アクティブ入力を持つ頻度を設定するために使用されます。
* **`dutyCyclePeriod`** ：デューティサイクルの計算に使用する期間。値が大きいほど、ブーストの変化に対応するのに時間がかかります。
* **`boostStrength`**：ブーストの強さを制御するために使用されます。ブーストの強さは boostStrength の関数として増加します。
* **`seed`** ：乱数発生器のシード。seed が &lt; 0 の場合、ランダムに生成されたシードが使用されます。
* **`spVerbosity`**：spVerbosity level（SPの冗長性レベル）: 0、1、2または 3で指定する。
* **`wrapAround`**：入力をミニカラムにマッピングする目的で、入力次元の先頭と末尾の入力を隣人とみなすかどうかを決定するブール値。

**メソッド**

**`compute(...)`**：

SpatialPooler クラスの主要なメソッドであり、入力 SDR を受け取り、出力される有効な列のセットを計算します。`learn` が True に設定されている場合、このメソッドは学習も行います。

