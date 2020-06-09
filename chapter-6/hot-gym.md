---
description: hot gym
---

# HotGym

HTMのサンプルアプリケーションHotGymについて解説します。時系列ごとの電気使用量から異常検知をおこないます。HTMのアルゴリズムを包括的に理解出るアプリなので、このアプリの実装方法が理解できればHTMシステムを活用することができます。

{% code title="python" %}
```python
import csv
import datetime
import os
import numpy as np
import random
import math

from htm.bindings.sdr import SDR, Metrics
from htm.encoders.rdse import RDSE, RDSE_Parameters
from htm.encoders.date import DateEncoder
from htm.bindings.algorithms import SpatialPooler
from htm.bindings.algorithms import TemporalMemory
from htm.algorithms.anomaly_likelihood import AnomalyLikelihood 
#FIXME 代わりにTM.anomalyを使用しますが、 py.AnomalyLikelihoodよりも悪い結果になります
from htm.bindings.algorithms import Predictor
```
{% endcode %}

入力ファイルを読み込みます。

{% code title="python" %}
```python
records = []
with open(_INPUT_FILE_PATH, "r") as fin:
    reader = csv.reader(fin)
    headers = next(reader)
    next(reader)
    next(reader)
    for record in reader:
      records.append(record)
```
{% endcode %}

日付エンコーダー・スカラエンコーダーを作成します。

{% code title="python" %}
```python
dateEncoder = DateEncoder(timeOfDay= (30, 1), weekend  = 21) 

scalarEncoderParams            = RDSE_Parameters()
scalarEncoderParams.size       = 700
scalarEncoderParams.sparsity   = 0.02
scalarEncoderParams.resolution = 0.88
scalarEncoder = RDSE( scalarEncoderParams )
encodingWidth = (dateEncoder.size + scalarEncoder.size)
enc_info = Metrics( [encodingWidth], 999999999 )
```
{% endcode %}

SPを定義します。

{% code title="python" %}
```python
sp = SpatialPooler(
    inputDimensions            = (encodingWidth,),
    columnDimensions           = (1638,),
    potentialPct               = 0.85,
    potentialRadius            = encodingWidth,
    globalInhibition           = True,
    localAreaDensity           = 0.04395604395604396,
    synPermInactiveDec         = 0.006,
    synPermActiveInc           = 0.04,
    synPermConnected           = 0.13999999999999999,
    boostStrength              = 3.0,
    wrapAround                 = True
)
sp_info = Metrics( sp.getColumnDimensions(), 999999999 )
```
{% endcode %}

TMを定義します。

{% code title="python" %}
```python
tm = TemporalMemory(
    columnDimensions          = (1638,), #sp.columnDimensions
    cellsPerColumn            = 13,
    activationThreshold       = 17,
    initialPermanence         = 0.21,
    connectedPermanence       = 0.13999999999999999, #sp.synPermConnected
    minThreshold              = 10,
    maxNewSynapseCount        = 32,
    permanenceIncrement       = 0.1,
    permanenceDecrement       = 0.1,
    predictedSegmentDecrement = 0.0,
    maxSegmentsPerCell        = 128,
    maxSynapsesPerSegment     = 64
)
tm_info = Metrics( [tm.numberOfCells()], 999999999 )
```
{% endcode %}

異常検知を定義します。尤度（ゆうど）はNABからコピーされます。

{% code title="python" %}
```python
probationaryPeriod = int(math.floor(float(0.1)*len(records)))
learningPeriod     = int(math.floor(probationaryPeriod / 2.0))
anomaly_history = AnomalyLikelihood(learningPeriod= learningPeriod,
                                  estimationSamples= probationaryPeriod - learningPeriod,
                                  reestimationPeriod= 100)

predictor = Predictor( steps=[1, 5], alpha=0.1)
predictor_resolution = 1
```
{% endcode %}

データセット内のすべてのデータを反復処理し、入力と出力を記録します.

{% code title="python" %}
```python
inputs      = []
anomaly     = []
anomalyProb = []
predictions = {1: [], 5: []}
```
{% endcode %}

入力データをもとに各アルゴリズムを実行します。

{% code title="python" %}
```python
for count, record in enumerate(records):

    # 日付文字列をPythonの日付オブジェクトに変換する.
    dateString = datetime.datetime.strptime(record[0], "%m/%d/%y %H:%M")
    # データ値の文字列をfloatに変換.
    consumption = float(record[1])
    inputs.append( consumption )

    # エンコーダを呼び出して、各値のビット表現を作成します。 これらは SDR オブジェクトです.
    dateBits        = dateEncoder.encode(dateString)
    consumptionBits = scalarEncoder.encode(consumption)

    # これらのすべてのエンコーディングを 1 つの大きなエンコーディングに連結して、
    # 空間プーリング用に使用します.
    encoding = SDR( encodingWidth ).concatenate([consumptionBits, dateBits])
    enc_info.addData( encoding )

    # これは、以下の計算方法で作成されます
    # Spatial Poolerと同じ寸法でなければなりません
    activeColumns = SDR( sp.getColumnDimensions() )

    # 入力空間に対する空間プーリングアルゴリズムの実行.
    sp.compute(encoding, True, activeColumns)
    sp_info.addData( activeColumns )

    # アクティブなミニカラムに対してTemporal Memoryアルゴリズムを実行する.
    tm.compute(activeColumns, learn=True)
    tm_info.addData( tm.getActiveCells().flatten() )

    # 何が起こるかを予測し、今起こったことに基づいて予測器を訓練する.
    pdf = predictor.infer( tm.getActiveCells() )
    for n in (1,5):
        if pdf[n]:
            predictions[n].append( np.argmax( pdf[n] ) * predictor_resolution )
        else:
            predictions[n].append(float('nan'))
        
    anomalyLikelihood = anomaly_history.anomalyProbability( consumption, tm.anomaly )
    
    anomaly.append( tm.anomaly )
    anomalyProb.append( anomalyLikelihood )

    predictor.learn(count, tm.getActiveCells(), int(consumption / predictor_resolution))
```
{% endcode %}

![&#x56F3;6-2](../.gitbook/assets/6-2.png)

