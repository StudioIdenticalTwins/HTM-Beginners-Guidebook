---
description: Classifier algorithms Tutori
---

# 分類アルゴリズムの実装方法

分類アルゴリズムを実装します。分類アルゴリズムを使うためには、ラベルとデータのセットが必要になります。ここでは "cat", "dog", "monkey", "slow loris" の4つのラベルを用意します。

{% code title="python3" %}
```python
import numpy as np
from htm.bindings.encoders import ScalarEncoder, ScalarEncoderParameters
from htm.bindings.sdr import SDR
from htm.algorithms import SpatialPooler as SP
from htm.bindings.algorithms import Classifier

categories = {"cat":0, "dog":1, "monkey":2, "slow loris":3}

scalarEncoderParams = ScalarEncoderParameters()
scalarEncoderParams.minimum = 0
scalarEncoderParams.maximum = 4
scalarEncoderParams.activeBits = 3
scalarEncoderParams.category = True

enc = ScalarEncoder(scalarEncoderParams)
```
{% endcode %}



{% code title="python3" %}
```python
inputSDR  = SDR( dimensions = (15, ) )
activeSDR = SDR( dimensions = (256, ) )
sp = SP(inputDimensions  = inputSDR.dimensions,
        columnDimensions = activeSDR.dimensions,
        localAreaDensity = 0.02,
        globalInhibition = True,
        seed             = 1,
        synPermActiveInc   = 0.01,
        synPermInactiveDec = 0.008)
```
{% endcode %}



{% code title="python3" %}
```python
clsr = Classifier()
```
{% endcode %}



{% code title="python3" %}
```python
for i in range(3):
    for i in range(4):
        inputSDR=enc.encode(i)
        sp.compute(inputSDR, True, activeSDR)
        print("Active Outputs: ",activeSDR)
        clsr.learn( activeSDR, list(categories.values())[i]  )
        print("Classifier learn: ",list(categories)[i] )
```
{% endcode %}



{% code title="python3" %}
```python
catdog=[1,1,0,0,0,1,0,0,0,0,0,0,0,0,0]
inputSDR = SDR(dimensions = (15, ))
inputSDR.dense = np.array(catdog)
print(inputSDR)
```
{% endcode %}



{% code title="python3" %}
```python
sp.compute(inputSDR, False, activeSDR)
print("Active Outputs: ",activeSDR)
pdf = clsr.infer( activeSDR )
print("PDF: ",pdf)
predict = list(categories)[np.argmax( pdf )]  
print("predict index: ",predict)
```
{% endcode %}

