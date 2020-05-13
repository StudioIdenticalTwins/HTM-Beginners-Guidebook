---
description: RDSE
---

# ランダム分散スカラーエンコーダ \(RDSE\)

ランダム分散スカラーエンコーダ（RDSE）は、数値スカラー（浮動小数点）値をSDRにエンコードします。 RDSE は ScalarEncoder よりも柔軟性があります。

このエンコーダは、入力範囲の最小値と最大値を知る必要がありません。

これは、構築時に input-&gt;output マッピングを割り当てません。 その代わり、エンコーディングは実行時に決定されます。

これにより、以前のエンコーディングと競合しないランダムなアクティブビットのセットを保証することができました。

また、エンコーダは SDR を作成した可能性の高い入力値にデコードすることができます。

このRDSEは、入力とアクティブビット間の関連付けを保存しません。 この方が高速で、使用するメモリも少なくて済みます。

#### パラメーター

* **`activeBits`**：エンコードされた出力 SDR の真のビット数です。
* **`category`**：入力が列挙されたカテゴリであることを意味します。true の場合、このエンコーダは符号なし整数のみをエンコードし、すべての入力は一意で重複しない表現になります。
* **`radius`**：半径以上で区切られた2つの入力は、重複しない表現になります。
* **`resolution`**：解像度（オンビットの密度）以上で区切られた2つの入力は、異なる表現を持つことが保証されています。
* **`seed`**：入力と他のすべてのパラメータが同じであっても、異なるエンコーダが異なる出力を生成することを強制します。 同じシード、パラメータ、入力を持つ 2 つのエンコーダは、同じ出力を生成します。
* **`size`**：エンコードされた出力 SDR の総ビット数です。
* **`sparsity`**：これはアクティブビット数を指定する別の方法です。sparsityの場合はサイズも指定する必要があります。activeBitsかsparsityのどちらか一方のみを指定してください。

実際に数値データをエンコードしてみましょう。以下のサンプルでは、24ビットのSDR上に3つのオンビットで数値を表現します。

入力時にマッピングを生成するため、最小値もしくは最大値のパラメータ設定が必要ないことに注意してください。

ここでは **`resolution`** の設定をし、以前の入力と次の入力が重なることを防ぎます。

{% code title="python3" %}
```python
from htm.bindings.encoders import RDSE, RDSE_Parameters
from htm.bindings.sdr import SDR

RDSE_Parameters = RDSE_Parameters()
RDSE_Parameters.activeBits = 3
RDSE_Parameters.resolution = 5
RDSE_Parameters.size       = 24

enc = RDSE( RDSE_Parameters )
```
{% endcode %}

エンコーダーのパラメータの指定ができました。特定の数値をエンコードします。

{% code title="python3" %}
```python
print ("3 =", enc.encode(3))
print ("4 =", enc.encode(4))
print ("5 =", enc.encode(5))
```
{% endcode %}

エンコードされたSDRは以下のようになります。スカラエンコーダと異なり、オンビットが連続していないことに注目してください。

{% code title="bash" %}
```python
3 = SDR( 24 ) 3, 19, 20
4 = SDR( 24 ) 3, 19, 20
5 = SDR( 24 ) 2, 3, 19
```
{% endcode %}

Numpy配列に形式を変換してみます。

{% code title="python3" %}
```python
print ("3 =", enc.encode(3).dense)
print ("4 =", enc.encode(4).dense)
print ("5 =", enc.encode(5).dense)
```
{% endcode %}

{% code title="terminal" %}
```bash
3 = [0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0]
4 = [0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0]
5 = [0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0]
```
{% endcode %}

比較的大きな数値を入力してみます。

{% code title="python3" %}
```python
print ("100  =", enc.encode(100))
print ("1000 =", enc.encode(1000))
```
{% endcode %}

{% code title="bash" %}
```bash
100 =  SDR( 24 ) 2, 6, 10
1000 = SDR( 24 ) 4, 12, 15
```
{% endcode %}

スカラエンコーディングと異なり、入力時にマッピングがおこなわれていることを確認してください。

