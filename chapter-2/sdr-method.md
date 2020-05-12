---
description: Method of SDR
---

# SDRのメソッド

#### `sdr.getSparsity()` 

SDR の sparsity \(SDR の全ビット数のうち、真のビット数の割合\) を計算します。

#### `sdr.getOverlap( sdr )` 

両方のSDRに共通する真のビット数を計算します。

#### `sdr.randomize( sparsity )` 

ランダムな SDR を作成し、SDR の現在の値を上書きします。 結果として、一様にランダムな活性化が得られます。

#### `dr.addNoise( percentNoise )` 

アクティブビットの一部をに移動させることで、SDR にノイズを追加します 。 この方法では、SDR の分散度は変化しませんが、真の値の位置を移動します。 結果として得られる SDR は、元の値との重複量が制御されます。

#### `sdr.concatenate( sdr_A, sdr_B )` 

複数のSDRから座標を取得し、その結果をこのSDRに格納します。

#### `sdr.intersection( sdr_A, sdr_B )` 

このメソッドは、各入力 SDR のアクティブビットの集合交点を計算します。

#### `sdr.reshape( new_dimensions )` 

SDR の次元を変更します。 合計サイズは変更しないでください。

#### `sdr.flatten()`

SDRの次元を1次元に変更します。

