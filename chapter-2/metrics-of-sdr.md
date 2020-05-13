---
description: Metrics of SDR
---

# SDRのメトリクス

**`htm.bindings.sdr.Sparsity`**

SDRの分散度を測定します。 指数平滑移動平均を使用して測定値を蓄積し、結果の要約を出力します。

引数 `period` は指数平滑移動平均の時定数（time constant）です。

* `Min` 最小値
* `Mean` 平均値
* `Std` 標準偏差値（standard deviation）
* `Max` 最大値

**`htm.bindings.sdr.ActivationFrequency`**

SDR の各値の活性化周波数を測定します。 指数平滑移動平均を用いて測定値を蓄積し、結果の要約を出力します。

活性化頻度は、\[0, 1\] の範囲の実数で、0 はアクティブにならないことを示し、1 は常にアクティブであることを示します。

引数 `period` は指数平滑移動平均の時定数（time constant）です。

* `Min` 最小値
* `Mean` 平均値
* `Std` 標準偏差値（standard deviation）
* `Max` 最大値
* `Entropy` エントロピー

**`htm.bindings.sdr.Overlap`**

SDRへの連続した割り当て間の重なりを測定します。 このクラスは、指数移動平均を使用して測定値を蓄積し、結果の要約を出力します。

このクラスは、アクティブな値の数で割ることで、オーバーラップを \[0, 1\] の範囲に正規化します。

引数 `period` は指数平滑移動平均の時定数（time constant）です。

* `Min` 最小値
* `Mean` 平均値
* `Std` 標準偏差値（standard deviation）
* `Max` 最大値

