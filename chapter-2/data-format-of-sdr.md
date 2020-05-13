---
description: data format of SDR
---

# SDRのデータ形式

SDRはNumpy形式の配列で表現され、3つのデータ形式をもちます。

1. **`dense`** - 高密度のデータ形式
2. **`sparse`** - 疎状態のデータ形式
3. **`coordinates`** - 座標を示すデータ形式

データは自動的に要求された形式に変換されキャッシュされるため、1 つの形式で何度も値を取得しても、追加のパフォーマンスコストは発生しません。

実際にサンプルコードをみてみましょう。

{% code title="pyhon3" %}
```python
import numpy as np # SDRはnumpy配列を使用します
from htm.bindings.sdr import SDR

# 9つの値を持つSDRを作成し、(3 x 3)グリッドに配置する
X = SDR(dimensions = (3, 3))

# この3つの記述は等価です
X.dense  = np.array([[0, 1, 0], [0, 1, 0], [0, 0, 1]])
X.sparse = np.array([ 1, 4, 8 ])
X.coordinates = np.array([[0, 1, 2], [1, 1, 2]])
```
{% endcode %}

ここでは\( 3 x 3 \)の2次元配列を配置していますが、SDRは何次元のデータも処理できます。データの代入前に`SDR` メソッドをつかい、初期化します。

その後それぞれのデータ形式にあわせて代入します。`dense` はオンビットとオフビットを両方表現し、`sparse` はオフビットの位置のみを表現しています。また `coordinates` はオンビットの座標を表現しています。

これらにより計算資源を削減できます。

{% code title="python3" %}
```python
# 任意の形式でデータにアクセスした場合、SDRは自動的にデータ形式を変換します。
X.dense ==  np.array([[0, 1, 0], [0, 1, 0], [0, 0, 1]])
X.sparse == np.array([ 1, 4, 8 ])
X.coordinates == np.array( [[0, 1, 2], [1, 1, 2]])
```
{% endcode %}

どのデータ形式であっても代入が可能です。

{% code title="python3" %}
```python
# データフォーマット変換はキャッシュされ, SDRの値が変更されるとキャッシュはクリアされます.
X.sparse = [1, 2, 3] # 新しいデータをSDRに割り当て、キャッシュをクリアする。
X.dense     # この行はフォーマットを変換します
X.dense     # この行は前の行の結果を再利用します.
```
{% endcode %}

データの入力がなければ、データの形式はキャッシュされます。

