---
description: Predictor algorithms tutorial
---

# 予測アルゴリズムの実装方法

予測アルゴリズムを実装します。予測アルゴリズムを使うためには、時系列ごとのラベルとデータのセットが必要になります。

ここでは300ステップで3周期となるsine波を用意します。

{% code title="python" %}
```python
import numpy as np

rows = 300
sine_wave={}

for i in range(rows):
    angle = (i * np.pi) / 50.0
    sine_value = np.sin(angle)
    sine_wave[i]=round(sine_value,2)
print(sine_wave)
```
{% endcode %}

