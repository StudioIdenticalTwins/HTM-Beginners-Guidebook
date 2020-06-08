---
description: Install RaspberryPi4 + 64 bit Ubuntu 18.04.4 LTS
---

# ARM64

![&#x56F3;1-4](../../.gitbook/assets/1-.png)

組み込み機器などで利用されてきたARM64にインストールできます。ただし64bitOSであることが必要要件となります。ここではRaspberry Pi OS \(64 bit\) beta test version を入れたものを使用します。Python のバージョンは3.7.3 になります。

ARM64環境では、ビルドをおこなう前に環境変数の指定が必要になります。.bashrc に以下のように環境変数を指定してください。

{% code title=".bashrc" %}
```text
export ARCHFLAGS="-arch arm64"
```
{% endcode %}

それ以外は、通常のUbuntuとおなじインストール方法をおこないます。ただし、コンパイラー依存関係が少し複雑なのでここでは説明を省きます。詳細は [**htm-beginners-book-1-1.ipynb**](https://github.com/StudioIdenticalTwins/HTM-Beginners-Notebook/blob/master/chapter-1/htm-beginners-book-1-1.ipynb) を参照してください。

