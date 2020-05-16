---
description: TM tutorial
---

# TMの実装方法

TMの実装方法をためします。ライブラリを読み込み、パラメータを指定し、SDRのデータ構造を作成します。

基本的な実装方法は、SPで出力されたアクティブなSDRを入力します。SPの出力とTMの入力の次元は一致しなければなりません。

{% code title="python3" %}
```python
from htm.bindings.sdr import SDR
from htm.algorithms import TemporalMemory as TM

activeSDR = SDR( dimensions = (50,) )
tm = TM(columnDimensions = activeSDR.dense.shape,
        cellsPerColumn=1,
        initialPermanence=0.5,
        connectedPermanence=0.5,
        minThreshold=8,
        maxNewSynapseCount=20,
        permanenceIncrement=0.1,
        permanenceDecrement=0.0,
        activationThreshold=8,
        )
```
{% endcode %}

入力データとして、ここでは仮にA・B・C・D・E を表すSDRを生成します。オンビット10で5つのカテゴリーなので、SDRのサイズは50ビットになります。

{% code title="python3" %}
```python
dataset = { inp : SDR( tm.numberOfColumns() ) for inp in "ABCDE" }

dataset['A'].dense[0:10]  = 1 
dataset['B'].dense[10:20] = 1 
dataset['C'].dense[20:30] = 1
dataset['D'].dense[30:40] = 1 
dataset['E'].dense[40:50] = 1 

for z in dataset.values():
  z.dense = z.dense
for inp in "ABCDE":
  print("Input:", inp, " Bits:", dataset[inp] )
```
{% endcode %}

入力の各SDRは以下のようになります。

{% code title="terminal" %}
```bash
Input: A  Bits: SDR( 50 ) 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
Input: B  Bits: SDR( 50 ) 10, 11, 12, 13, 14, 15, 16, 17, 18, 19
Input: C  Bits: SDR( 50 ) 20, 21, 22, 23, 24, 25, 26, 27, 28, 29
Input: D  Bits: SDR( 50 ) 30, 31, 32, 33, 34, 35, 36, 37, 38, 39
Input: E  Bits: SDR( 50 ) 40, 41, 42, 43, 44, 45, 46, 47, 48, 49
```
{% endcode %}

続いて、**`tm.compute()`** メソッドを使って時間プーラーを使って学習・予測をおこないます。

for ループをつかい、各SDRを1回づつ入力します。

{% code title="python3" %}
```python
for inp in "ABCDE": 
    activeColumns = dataset[inp]
    tm.compute(activeColumns, learn = True)
    tm.activateDendrites(True)
    print(tm.getPredictiveCells())
```
{% endcode %}

TMはセル間の結びつきを学習・予測するため、SDRの出力はありません。予測セルを出力するには2つのメソッドが必要となります。

**`tm.activateDendrites()`**でアクティブな接続を呼び出し、**`tm.getPredictiveCells()`** で予測セルを取得します。

1回目の予測セルを確認してみましょう。

{% code title="terminal" %}
```bash
SDR( 50, 1 )
SDR( 50, 1 )
SDR( 50, 1 )
SDR( 50, 1 )
SDR( 50, 1 )
```
{% endcode %}

時間プーラーの学習では、ミニカラムがはじめてのデータを参照する際、それらは予測にないためバースト（Burst）が発生します。つまり、ミニカラムのすべてのセルがそのシーケンスを学習しようとします。

そのため予測がおこなわれていないことに注目してください。もう一度同じデータを入力します。

{% code title="python3" %}
```python
for inp in "ABCDE": 
    activeColumns = dataset[inp]
    tm.compute(activeColumns, learn = True)
    tm.activateDendrites(True)
    print(tm.getPredictiveCells())
```
{% endcode %}

2回目の予測セルを確認してみましょう。

{% code title="terminal" %}
```bash
SDR( 50, 1 ) 10, 11, 12, 13, 14, 15, 16, 17, 18, 19
SDR( 50, 1 ) 20, 21, 22, 23, 24, 25, 26, 27, 28, 29
SDR( 50, 1 ) 30, 31, 32, 33, 34, 35, 36, 37, 38, 39
SDR( 50, 1 ) 40, 41, 42, 43, 44, 45, 46, 47, 48, 49
SDR( 50, 1 ) 0, 1, 2, 3, 4, 5, 6, 7, 8, 9
```
{% endcode %}

1回目の入力データをもとに予測がおこなわれていることが分かります。

### 接続状況の確認

TMは入力データをもとに、セグメントを通じてセル間の接続度を更新します。処理の内部情報となる入力と出力の接続状況は `print()` で確認することができます。

初期状態のTMは以下のような状態でした。

{% code title="ptyhon3" %}
```python
print(tm)
```
{% endcode %}

{% code title="terminal" %}
```bash
Temporal Memory Connections:
    Inputs (0) ~> Outputs (50) via Segments (0)
    Segments on Cell Min/Mean/Max 0 / 0 / 0
    Potential Synapses on Segment Min/Mean/Max 4294967295 / nan / 0
    Connected Synapses on Segment Min/Mean/Max 65535 / nan / 0
    Synapses Dead (nan%) Saturated (nan%)
    Synapses pruned (nan%) Segments pruned (nan%)
```
{% endcode %}

2回データ入力が終わったあとの内部情報を確認してみましょう。

{% code title="terminal" %}
```bash
Temporal Memory Connections:
    Inputs (50) ~> Outputs (50) via Segments (50)
    Segments on Cell Min/Mean/Max 1 / 1 / 1
    Potential Synapses on Segment Min/Mean/Max 10 / 10 / 10
    Connected Synapses on Segment Min/Mean/Max 10 / 10 / 10
    Synapses Dead (0%) Saturated (0%)
    Synapses pruned (0%) Segments pruned (0%)
```
{% endcode %}

シナプスの数値変化に注目してください。

