---
description: Overview of HTM
---

# HTM概要

ホーキンスが『考える脳考えるコンピューター』執筆した後、構築したシステムは階層的一時記憶（以下HTM）と名付けられました。

今回考察するのは、第二世代アルゴリズムになります。主にNumenta社の公開している情報を参照します。

HTMは、新皮質の領域（リージョン/Region）の構造をもとにしています。旧脳は含まれず、脳全体を再現したものでないことに注意してください。

新皮質はどの領域でも均一にはたらきますので（マウントキャッスルの発見）、ひとつの領域は「記憶による予測」をおこなう最小単位として機能します。

HTMのリージョン（Region）は新皮質の柱状構造を模しており、ミニカラム（Mini-Column）の集合（Mini-Columns）からできています。ミニカラムは複数のセル（Cell）をもちます。

HTMのセルは、新皮質の錐体細胞の機能を再現します。「入力データ」をフィードフォワード（Feedfoward）「普遍の表現」をフィードバック（Feedback）として受け取ります。

入力データは視床を通じて第1層に戻り、領域全体に広がります。それをもとに新皮質は「自己連想記憶」を呼び出しました。HTMセルは、シナプスの結合状態から「文脈」（Context）をとらえ、「自己連想記憶」を呼び出します。

