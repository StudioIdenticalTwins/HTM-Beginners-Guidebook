---
description: Category Encoder
---

# カテゴリーエンコーダー

入力のカテゴリをエンコードするには、ScalarEncoderまたはRandom Distributed Scalar Encoder \(RDSE\)を作成し、パラメータ `category=True` を設定します。 

そして、エンコードする前にカテゴリを整数に列挙します。

簡単のため、カテゴリーの各要素の順番を数値に置き換えて、実際にやってみましょう。要素0は"unknown"のために予約しておきます。

ここではスカラーエンコーダーよ使用します。パラメーターのカテゴリーを `True` で指定します。パラメータの最小値と最大値によってカテゴリーの数が決定されます。

{% code title="python3" %}
```python
categories = ("unknown","cat", "dog", "monkey", "slow loris")

scalarEncoderParams = ScalarEncoderParameters()
scalarEncoderParams.minimum = 0
scalarEncoderParams.maximum = 5
scalarEncoderParams.activeBits = 3
scalarEncoderParams.category = True

enc = ScalarEncoder(scalarEncoderParams)
```
{% endcode %}

エンコーダーのパラメータの指定ができました。カテゴリーの要素に合わせた数値をエンコードします。

{% code title="python3" %}
```python
print ("cat =       ", enc.encode(1))
print ("dog =       ", enc.encode(2))
print ("monkey =    ", enc.encode(3))
print ("slow loris =", enc.encode(4))
```
{% endcode %}

エンコードされたSDRは以下のようになります。パラメータの最小値と最大値、そしてオンビット数を指定したので自動的にSDRサイズがエンコードされます。

{% code title="bash" %}
```python
cat =        SDR( 18 ) 3, 4, 5
dog =        SDR( 18 ) 6, 7, 8
monkey =     SDR( 18 ) 9, 10, 11
slow loris = SDR( 18 ) 12, 13, 14
```
{% endcode %}

Numpy配列に形式を変換してみます。

{% code title="python3" %}
```python
print ("cat =       ", enc.encode(1).dense)
print ("dog =       ", enc.encode(2).dense)
print ("monkey =    ", enc.encode(3).dense)
print ("slow loris =", enc.encode(4).dense)
```
{% endcode %}

{% code title="terminal" %}
```bash
cat =        [0 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0]
dog =        [0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0]
monkey =     [0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0]
slow loris = [0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0]
```
{% endcode %}



