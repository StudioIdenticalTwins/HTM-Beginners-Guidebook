---
description: anomaly likelihood
---

# 異常検知

このモジュールは、与えられたモデルから平均化された異常スコアの分布を分析し、推定します。新しい異常スコアsが与えられると、P\(score &gt;= s\)を推定します。数P\(score &gt;= s\)は、予測可能性の現在の状態の尤度を表します。

例えば、尤度が0.01または1%であれば、100レコードに1つの割合でこれだけの予測可能性があることを意味します。この数字は見かけほど異常ではありません。

1分ごとに到着するレコードの場合、これは1時間40分に1回ということになります。可能性が0.0001または0.01%というのは、10,000レコードのうち1回、つまり7日に1回程度見られることを意味します。

コードを使用するには2つの方法があります。ひとつめはヘルパークラス  **`AnomalyLikelihood()`** 使用する方法。ふたつめは個々の関数**`estimateAnomalyLikelihoods()`** と**`updateAnomalyLikelihoods()`** を使用する方法です。

### クラス <a id="&#x30AF;&#x30E9;&#x30B9;"></a>

**`AnomalyLikelihood(claLearningPeriod=None, learningPeriod=288, estimationSamples=100, historicWindowSize=8640, reestimationPeriod=100)`**

{% code title="python" %}
```python
anomalyLikelihood = AnomalyLikelihood()
while still_have_data:
    # Get anomaly score from model
    
    # Compute probability that an anomaly has ocurred
    anomalyProbability = anomalyLikelihood.anomalyProbability(
        value, anomalyScore, timestamp)
```
{% endcode %}

**`claLearningPeriod`** は非推奨の名前ですが、 **`claLearningPeriod`**と**`learningPeriod`** は同じ変数を指定しています。

* **`learningPeriod`**: アルゴリズムがデータセットの基本的なパターンを学習し、異常スコアが「落ち着く」までに必要な反復回数を指定します。デフォルトは経験的な観測に基づいていますが、実際にはより複雑なドメインではより大きくなる可能性があります。これを大きくしすぎると、実際の異常は無視されてフラグが立てられなくなる可能性があるという欠点があります。
* **`estimationSamples`**: ガウシアンの初期推定に必要な妥当な異常値の数を指定します. デフォルトの100レコードは妥当です - ガウス分布のまともな推定値を得るのに十分なサンプルが必要なだけです。ガウス分布はデフォルトでは10回の反復ごとに再推定されるので、これを調整する必要はないでしょう。
* **`historicWindowSize`**: ガウス分布の周期的な再推定のために維持するための過去のデータポイントのスライディングウィンドウのサイズ. 注意: デフォルトの8640は、5分間隔で1ヶ月分の履歴に基づいています。
* **`reestimationPeriod`**: ガウス分布をどのくらいの頻度で再推定するかを指定します。理想的には繰り返しごとに再推定するのがよいのですが、これはパフォーマンスの低下につながります。一般的に，処理されるレコードの総数に比べてこの数値が小さい限り，システムはこの数値にあまり敏感ではありません

### 低レベル機能の使い方 <a id="&#x4F4E;&#x30EC;&#x30D9;&#x30EB;&#x6A5F;&#x80FD;&#x306E;&#x4F7F;&#x3044;&#x65B9;"></a>

主なインターフェース・ルーチンは2つあります。

* はじめにたまに、**`estimatedAnomalyLikelihoods()`**バッチ・ルーチン。
* 新しいデータポイントごとに、**`updateAnomalyLikelihoods()`** と呼ばれるオンラインルーチン。

最初に：

{% code title="python" %}
```python
 likelihoods, avgRecordList, estimatorParams = \
     estimateAnomalyLikelihoods(metric_data)
```
{% endcode %}

新しいデータを取得するたびに：

{% code title="python" %}
```python
likelihoods, avgRecordList, estimatorParams = \
    updateAnomalyLikelihoods(data2, estimatorParams)
```
{% endcode %}

また、再度（**`AnomalyLikelihoods`**を更新するために上記の呼び出しで返された新しい**`estimatorParams`**を使用していることを確認してください！）

{% code title="python" %}
```python
likelihoods, avgRecordList, estimatorParams = \
    updateAnomalyLikelihoods(data3, estimatorParams)
```
{% endcode %}

たまには、最近のデータをたくさん使って **`estimator`** を更新してくださいI

{% code title="python" %}
```python
likelihoods, avgRecordList, estimatorParams = \
    estimateAnomalyLikelihoods(lots_of_metric_data)
```
{% endcode %}

### パラメーター <a id="&#x30D1;&#x30E9;&#x30E1;&#x30FC;&#x30BF;&#x30FC;"></a>

上記の関数が返すパラメータdictは、以下のような構造になっています。注：クライアントはその詳細を知る必要はありません。

```text
 {
   "distribution":               # describes the distribution
     {
       "name": STRING,           # name of the distribution, such as 'normal'
       "mean": SCALAR,           # mean of the distribution
       "variance": SCALAR,       # variance of the distribution

       # There may also be some keys that are specific to the distribution
     },

   "historicalLikelihoods": []   # Contains the last windowSize likelihood
                                 # values returned

   "movingAverage":              # stuff needed to compute a rolling average
                                 # of the anomaly scores
     {
       "windowSize": SCALAR,     # the size of the averaging window
       "historicalValues": [],   # list with the last windowSize anomaly
                                 # scores
       "total": SCALAR,          # the total of the values in historicalValues
     },

 }
```



