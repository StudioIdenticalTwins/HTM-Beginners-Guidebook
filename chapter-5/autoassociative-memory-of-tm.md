---
description: Autoassociative memory of TM
---

# TMの自己連想記憶

TMは異なるシーケンスを、どのように処理するのでしょうか。いくつかの基本的な特性を確認します。

"ABCDXY"という6つのカテゴリーを用意します。シーケンス"ABCD"とシーケンス"XBCY"を与えることで学習・予測がどの様に変化するかを確認します。

{% code title="python3" %}
```python
import numpy as np
import random
random.seed(1)

from htm.bindings.sdr import SDR
from htm.algorithms import TemporalMemory as TM
```
{% endcode %}

{% code title="python3" %}
```python
tm = TM(
  columnDimensions = (2048,),
  cellsPerColumn=8,
  initialPermanence=0.21,
  connectedPermanence=0.3,
  minThreshold=15,
  maxNewSynapseCount=40,
  permanenceIncrement=0.1,
  permanenceDecrement=0.1,
  activationThreshold=15,
  predictedSegmentDecrement=0.01,
  )
```
{% endcode %}

"ABCDXY"のカテゴリーはSPの出力を想定して、大きさ2048ビットでスパース値0.02%のSDRで表現します。

{% code title="python3" %}
```python
sparsity   = 0.02
sparseCols = int(tm.numberOfColumns() * sparsity)
dataset    = {inp : SDR( tm.numberOfColumns() ) for inp in "ABCDXY"}
for i, inp in enumerate("ABCDXY"):
  dataset[inp].dense[ i * sparseCols : (i + 1) * sparseCols ] = 1
  dataset[inp].dense = dataset[inp].dense # この行は、SDR の高密度データが変更されたことを SDR に通知する.
  print("Input", inp, "is bits at indices: [",  i * sparseCols, '-', (i + 1) * sparseCols, ']')

seq1 = "ABCD"
seq2 = "XBCY"
seqT = "ABCDXY"
```
{% endcode %}

{% code title="terminal" %}
```bash
Input A is bits at indices: [ 0 - 40 ]
Input B is bits at indices: [ 40 - 80 ]
Input C is bits at indices: [ 80 - 120 ]
Input D is bits at indices: [ 120 - 160 ]
Input X is bits at indices: [ 160 - 200 ]
Input Y is bits at indices: [ 200 - 240 ]
```
{% endcode %}

1）まずは、シーケンス"ABCD"を10回学習させ、"ABCDXY"を学習なしで1回づつ予測させます。

{% code title="python3" %}
```python
ts = 0  
x = []
y = []
for t in range(10):
    tm.reset()
    for inp in seq1:
        v = SDR(dataset[inp])
        tm.compute( v, learn=True)
        x.append(ts)
        y.append( 1 - tm.anomaly )
        ts += 1
```
{% endcode %}

{% code title="python3" %}
```python
for inp in sorted(dataset.keys()):
    print("--- " + inp + " ---")
    sdr = dataset[inp]
    tm.reset()
    tm.compute( sdr, learn=False)
    tm.activateDendrites(learn=False)
    activeColumnsIndices   = [tm.columnForCell(i) for i in tm.getActiveCells().sparse]
    predictedColumnIndices = [tm.columnForCell(i) for i in tm.getPredictiveCells().sparse]
    print("Active cols: " + str(sorted(set(activeColumnsIndices))))
    print("Predicted cols: " + str(sorted(set(predictedColumnIndices))))
    print("")
```
{% endcode %}

{% code title="terminal" %}
```bash
--- A ---
Active cols: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39]
Predicted cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]

--- B ---
Active cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]
Predicted cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]

--- C ---
Active cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]
Predicted cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159]

--- D ---
Active cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159]
Predicted cols: []

--- X ---
Active cols: [160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199]
Predicted cols: []

--- Y ---
Active cols: [200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239]
Predicted cols: []
```
{% endcode %}

X、Yの予測がされていないことを確認してください。

2）つづいてまずは、シーケンス"XBCY"を10回学習させます。**`tm.reset()`** で学習結果をリセットしますので注意してください。その後"ABCDXY"を学習なしで1回づつ予測させます。

{% code title="python3" %}
```python
ts = 0  
x = []
y = []
for t in range(10):
    tm.reset()
    for inp in seq2:
        v = SDR(dataset[inp])
        tm.compute( v, learn=True)
        x.append(ts)
        y.append( 1 - tm.anomaly )
        ts += 1
```
{% endcode %}

{% code title="python3" %}
```python
for inp in sorted(dataset.keys()):
    print("--- " + inp + " ---")
    sdr = dataset[inp]
    tm.reset()
    tm.compute( sdr, learn=False)
    tm.activateDendrites(learn=False)
    activeColumnsIndices   = [tm.columnForCell(i) for i in tm.getActiveCells().sparse]
    predictedColumnIndices = [tm.columnForCell(i) for i in tm.getPredictiveCells().sparse]
    print("Active cols: " + str(sorted(set(activeColumnsIndices))))
    print("Predicted cols: " + str(sorted(set(predictedColumnIndices))))
    print("")
```
{% endcode %}

{% code title="terminal" %}
```bash
--- A ---
Active cols: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39]
Predicted cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]

--- B ---
Active cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]
Predicted cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]

--- C ---
Active cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]
Predicted cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239]

--- D ---
Active cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159]
Predicted cols: []

--- X ---
Active cols: [160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199]
Predicted cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]

--- Y ---
Active cols: [200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239]
Predicted cols: []
```
{% endcode %}

D、Yの予測がされていないことを確認してください。

3）最後に、シーケンス"ABCD"とシーケンス"XBCY"をランダムに1,000回学習させます。そして"ABCDXY"を学習なしで1回づつ予測させます。

{% code title="python3" %}
```python
for t in range(1000):
  seq = random.choice([ seq1, seq2 ])
  for inp in seq:
    tm.compute( dataset[inp], learn=True)
```
{% endcode %}

{% code title="python3" %}
```python
for inp in sorted(dataset.keys()):
    print("--- " + inp + " ---")
    sdr = dataset[inp]
    tm.reset()
    tm.compute( sdr, learn=False)
    tm.activateDendrites(learn=False)
    activeColumnsIndices   = [tm.columnForCell(i) for i in tm.getActiveCells().sparse]
    predictedColumnIndices = [tm.columnForCell(i) for i in tm.getPredictiveCells().sparse]
    print("Active cols: " + str(sorted(set(activeColumnsIndices))))
    print("Predicted cols: " + str(sorted(set(predictedColumnIndices))))
    print("")
```
{% endcode %}

{% code title="terminal" %}
```bash
--- A ---
Active cols: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39]
Predicted cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]

--- B ---
Active cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]
Predicted cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]

--- C ---
Active cols: [80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119]
Predicted cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239]

--- D ---
Active cols: [120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159]
Predicted cols: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199]

--- X ---
Active cols: [160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199]
Predicted cols: [40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79]

--- Y ---
Active cols: [200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212, 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228, 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239]
Predicted cols: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180, 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196, 197, 198, 199]
```
{% endcode %}

ここではカテゴリー"C"に注目してください。予測したミニカラムは"D"と"Y"を内在した表現になっています。

