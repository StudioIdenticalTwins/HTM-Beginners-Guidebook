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

* **`columnDimensions`** ：ミニカラム空間の次元
* **`cellsPerColumn`**：ミニカラムあたりのセル数
* **`activationThreshold`**：あるセグメント上のアクティブな接続シナプスの数が少なくともこの閾値以上であれば、そのセグメントはアクティブになります。
* **`initialPermanence`**：新しいシナプスの初期の永続値。
* **`connectedPermanence`**：シナプスの永続値がこの値よりも大きい場合、シナプスは接続されていることになります。
* **`minThreshold`** ：あるセグメントで活動している潜在的なシナプスの数が少なくともこの閾値以上であれば、それは一致しており、学習の対象となります。
* **`maxNewSynapseCount`**：学習中にセグメントに追加されるシナプスの最大数。
* **`permanenceIncrement`** ：学習中にシナプスの永続値が増加する量。
* **`permanenceDecrement`**：学習中にシナプスの永続値が減少する量。
* **`predictedSegmentDecrement`**：不正確な予測に対してセグメントが罰せられる量。 良い値は、\(カラムレベルの分散度 × permanenceIncrement\)よりも少しだけ大きい値です。つまり、カラムレベルの分散度が2%で permanenceIncrementが0.01の場合、このパラメータは4% × 0.01 = 0.0004のような値になります。
* **`seed`**：乱数発生器の種。
* **`maxSegmentsPerCell`**：セルあたりのセグメントの最大数。
* **`maxSynapsesPerSegment`**：1セグメントあたりのシナプスの最大数。
* **`checkInputs`**：アクティブカラムが重複せずにソートされているかどうかをチェックするかどうか。これを無効にすると、少しだけ速度が上がります。
* **`externalPredictiveInputs`** ：外部予測入力の数。 これらの値はこのTMには関係なく、別の領域からの入力を表しています。 この TM は、この TemporalMemory の一部であるセルに加えて、これらの入力とシナプスを形成する。 この値が与えられた場合（0よりも大きい場合）、これらの外部入力のアクティブセルと勝者セルは、TM.computeとTM.activateDendritesメソッドに与えられなければならない。

**メソッド**

**`compute(...)`**：

時間記憶アルゴリズムの1タイムステップを実行します。 このメソッドは activateDendrites を呼び出し、次に activateCells を呼び出します。

TemporalMemoryをそのcomputeメソッドを介して使用することで、タイムステップの最後に常にgetActiveCellsを呼び出すことができるようになります。

