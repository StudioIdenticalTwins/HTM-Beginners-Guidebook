---
description: SP Tutorial
---

# SPの実装方法

まずは、SPの実装方法をためします。ライブラリを読み込み、パラメータを指定し、SDRのデータ構造を作成します。

{% code title="python3" %}
```python
from htm.bindings.sdr import SDR
from htm.algorithms import SpatialPooler as SP

inputSDR  = SDR( dimensions = (32, 32) )
activeSDR = SDR( dimensions = (64, 64) )
sp = SP(inputDimensions  = inputSDR.dimensions,
        columnDimensions = activeSDR.dimensions,
        localAreaDensity = 0.02,
        globalInhibition = True,
        seed             = 1,
        synPermActiveInc   = 0.01,
        synPermInactiveDec = 0.008)
```
{% endcode %}

続いて、SDRへエンコードしたデータを入力し、**`sp.compute()`** メソッドを使って空間プーラーへ変換します。

ここでは、**`sdr.randomize()`** メソッドをつかいランダムなSDRを生成します。

htm.coreの計算処理は、コア部分をのぞきPythonとNumpyでおこないます。ここではfor ループをつかい3回入力します。

{% code title="python3" %}
```python
for i in range(3):
    inputSDR.randomize( .02 )
    sp.compute(inputSDR, True, activeSDR)
    print("Active Outputs " + str(activeSDR))
    print("")
```
{% endcode %}

出力をみてみましょう。異なる入力に対し、異なる出力がされていることに注目してください。

{% code title="terminal" %}
```bash
Inputs: SDR( 32, 32 ) 33, 113, 121, 123, 131, 153, 183, 187, 311, 312, 379, 508, 530, 661, 696, 718, 758, 859, 900, 990
Outputs: SDR( 64, 64 ) 82, 87, 233, 295, 342, 391, 469, 470, 525, 538, 645, 966, 1009, 1022, 1058, 1088, 1135, 1155, 1169, 1180, 1197, 1358, 1365, 1426, 1606, 1645, 1775, 1895, 2027, 2124, 2251, 2253, 2299, 2325, 2330, 2443, 2540, 2629, 2987, 2999, 3006, 3031, 3071, 3085, 3090, 3105, 3125, 3164, 3202, 3217, 3243, 3291, 3346, 3403, 3420, 3442, 3447, 3503, 3558, 3567, 3608, 3610, 3674, 3681, 3686, 3714, 3731, 3760, 3788, 3855, 3856, 3875, 3882, 3907, 3921, 3967, 3968, 4003, 4053, 4060, 4071

Inputs: SDR( 32, 32 ) 27, 52, 72, 146, 162, 175, 181, 191, 196, 467, 513, 544, 546, 626, 785, 799, 890, 916, 937, 951
Outputs: SDR( 64, 64 ) 23, 157, 313, 427, 534, 633, 651, 667, 740, 771, 933, 1035, 1093, 1097, 1098, 1264, 1290, 1436, 1556, 1582, 1894, 1895, 1963, 1987, 2037, 2110, 2142, 2270, 2384, 2513, 2570, 2863, 2924, 2945, 3058, 3067, 3071, 3100, 3120, 3162, 3178, 3185, 3192, 3195, 3245, 3266, 3290, 3334, 3338, 3339, 3370, 3430, 3441, 3444, 3459, 3492, 3579, 3619, 3627, 3651, 3684, 3718, 3745, 3749, 3764, 3765, 3786, 3824, 3829, 3830, 3834, 3843, 3876, 3890, 3895, 3910, 3948, 3978, 3987, 4060, 4067

Inputs: SDR( 32, 32 ) 14, 18, 57, 159, 299, 328, 401, 411, 440, 499, 515, 545, 578, 662, 801, 812, 841, 902, 907, 984
Outputs: SDR( 64, 64 ) 11, 16, 65, 173, 273, 499, 510, 518, 574, 714, 731, 846, 893, 920, 989, 990, 1036, 1148, 1180, 1276, 1346, 1373, 1499, 1502, 1507, 1537, 1581, 1618, 1678, 1915, 1921, 1952, 2159, 2163, 2232, 2247, 2297, 2494, 2907, 2915, 2932, 2943, 2974, 2988, 3040, 3098, 3141, 3207, 3232, 3255, 3286, 3384, 3394, 3446, 3448, 3449, 3464, 3524, 3559, 3574, 3584, 3593, 3598, 3604, 3610, 3645, 3672, 3732, 3772, 3792, 3806, 3820, 3825, 3845, 3865, 3949, 4040, 4052, 4058, 4065, 4090
```
{% endcode %}

SPの主なつかい方は以上です。基本的に入力データを疎なSDRへ変換し、アクティブなセルをTMへ引き渡す目的で使用します。

### 接続状況の確認

SPは入力データをもとに、セグメントを通じてシナプスの接続度を更新します。処理の内部情報となる入力と出力の接続状況は `print()` で確認することができます。

初期状態のSPは以下のような状態でした。

{% code title="ptyhon3" %}
```python
print(sp)
```
{% endcode %}

{% code title="terminal" %}
```bash
Spatial Pooler Connections:
    Inputs (1024) ~> Outputs (4096) via Segments (4096)
    Segments on Cell Min/Mean/Max 1 / 1 / 1
    Potential Synapses on Segment Min/Mean/Max 512 / 512 / 512
    Connected Synapses on Segment Min/Mean/Max 218 / 256.142 / 301
    Synapses Dead (4.29153e-06%) Saturated (4.76837e-07%)
    Synapses pruned (0%) Segments pruned (0%)
```
{% endcode %}

3回データ入力が終わったあとの内部情報を確認してみましょう。

{% code title="terminal" %}
```bash
Spatial Pooler Connections:
    Inputs (1024) ~> Outputs (4096) via Segments (4096)
    Segments on Cell Min/Mean/Max 1 / 1 / 1
    Potential Synapses on Segment Min/Mean/Max 512 / 512 / 512
    Connected Synapses on Segment Min/Mean/Max 218 / 256.024 / 301
    Synapses Dead (0.00233603%) Saturated (1.14441e-05%)
    Synapses pruned (0%) Segments pruned (0%)
```
{% endcode %}

SPは未知の入力に対し、ブースティングをおこないより多くのシナプスを活性化させようとします。シナプスの数値変化に注目してください。

