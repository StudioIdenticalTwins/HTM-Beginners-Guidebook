---
description: document of htm.core
---

# ドキュメント

`htm.core`のドキュメントはPydocに格納されており、インタープリターからヘルプコマンドで呼び出せます。

{% code title="terminal" %}
```bash
python3
>>> help()
help> htm.bindings.sdr.SDR
```
{% endcode %}

閉じる時は `q` をタイプしてください。また、Pythonスクリプトでも開けます。

{% code title="terminal" %}
```bash
python3 -m pydoc htm.bindings.sdr.SDR
```
{% endcode %}

以下、主要なクラスを載せておきますので、必要に応じて確認してください。

### SDR

**`htm.bindings.sdr`**

* **`htm.bindings.sdr.SDR`**
* **`htm.bindings.sdr.Metrics`**
* **`htm.bindings.sdr.Sparsity`**
* **`htm.bindings.sdr.ActivationFrequency`**
* **`htm.bindings.sdr.Overlap`**

### Encoders

**`htm.bindings.encoders`** 

* **`htm.bindings.encoders.RDSE`**
* **`htm.bindings.encoders.RDSE_Parameters`** 
* **`htm.bindings.encoders.ScalarEncoder`** 
* **`htm.bindings.encoders.ScalarEncoderParameters`** 

{% hint style="warning" %}
**`htm.bindings.encoders.DateEncoder`**は古いドキュメントなので、`py/htm/encoders/date.py` のコメントを参照してください。
{% endhint %}

### SP

**`htm.algorithms.SpatialPooler`** 

### TM

**`htm.algorithms.TemporalMemory`** 

### Classifier algorithms

**`htm.bindings.algorithms.Classifier`** 

### Predictor algorithms

**`htm.bindings.algorithms.Predictor`** 

### anomaly likelihood

**`htm.algorithms.anomaly_likelihood`** 

