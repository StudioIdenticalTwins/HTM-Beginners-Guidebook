---
description: Install Ubuntu 18.04.4
---

# Ubuntu

![&#x56F3;1-1](../../.gitbook/assets/1-1.png)

**Ubuntu18.04**

[htm.core](https://github.com/htm-community/htm.core) の設置方法を参考に、プラットフォームをインストールします。コア部分はC++ですが、API経由でPython3で利用できます。 

Ubuntuは依存関係の影響を受けにくいため、最も簡単にビルドすることができます。ここではUbuntu18.04.4 LTS、Python 3.6.9でインストールします。

**Pythonによるビルド**

コア部分であるC++をコンパイルするために、cmake、g++、clang++ などが必要になります。もしインストールしていなければ事前にインストールしてください。

Python3で書かれたビルドスクリプトがあるので、リポジトリをクローンし、スクリプトを実行します。

{% code title="terminal" %}
```bash
git clone https://github.com/htm-community/htm.core.git
cd htm.core
python3 setup.py install
```
{% endcode %}

インストールが完了したらPython3のインタラクティブモードでライブラリをインポートができるか確認します。

{% code title="terminal" %}
```bash
python3
>>> import htm           # Python Library
>>> import htm.bindings  # C++ Extensions
>>> help( htm )          # Documentation
```
{% endcode %}

インポートが確認できたらテストスクリプトを実行します。

{% code title="terminal" %}
```bash
python3 setup.py test
```
{% endcode %}

