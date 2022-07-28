# Pythonをexeに変換する方法
Pythonコード（``.py``）をexeファイルに変換する方法

## 利用ツール
- 下記を利用した方法を説明する。
  - ``pyinstaller``: exe変換するための外部ライブラリ。
    - sklearnなどの外部ライブラリのパッケージはそのままexe化できない。exe化の前に、別途インポートが必要なサブモジュールを手動で設定しておく必要がある。
  - ``pipenv``: 仮想環境用の外部ライブラリ。``pip``（パッケージ管理）と``venv``（仮想環境構築）をまとめて使えるツール。
    - ローカル環境でexe化すると、exeファイルのサイズがかなり大きくなる。
    - exeファイルのサイズ削減のために、仮想環境上で必要なライブラリだけをインストールした後、exe化する。


## 手順
### 手順1：pipenvのインストール
- ローカル環境に``pipenv``をインストールする。

```cmd
[local] $ pip install pipenv
```

### 手順2：pythonコード実装
- 好きなディレクトリで作業用ディレクトリ``workdir``を作成する。
- ``workdir``内にexe化したいpythonファイル``execute.py``を作成し、コードを実装する。

```python
[workdir\execute.py]
import numpy
import scipy
import pandas
import matplotlib
import sklearn
...
``` 

### 手順3：パッケージインストール用ファイル作成
- 対象pythonファイル``execute.py``で使用するパッケージを``workdir\requiremenets.txt``に書く。
  - exe化のために``pyinstaller``は必須

```
[workdir\requiremenets.txt]
pyinstaller
numpy
scipy
pandas
matplotlib
sklearn
``` 

### 手順4：仮想環境構築
- コマンドプロンプトを起動し、currentディレクトリを``workdir``まで移動する。

````cmd
[local] $ cd workdir
````

- pipenvコマンドで、``workdir``内で仮想環境を構築（初期化）する。
  - ``workdir``内に仮想環境情報が記載されている``Pipfile``が作成される。

````cmd
[local] ~/workdir $ pipenv --python 3.9
````

- pipenvコマンドで、仮想環境内に``workdir\requiremenets.txt``に書いたパッケージをインストールする。
  - 通常のpip同様、proxy設定には注意。
  - 完了すると、``Pipfile.lock``が作成される。

````cmd
[local] ~/workdir $ pipenv install -r requirements.txt
````

### 手順5
- pipenvコマンドで、仮想環境内に移動する。

````cmd
[local] ~/workdir $ pipenv shell
(workdir) ~/workdir $ 
````

- ``pip list``コマンドで、仮想環境内にインストールされたパッケージを確認できる。

````shell
(workdir) ~/workdir $ pip list
Package                           Version
--------------------------------- ---------
matplotlib                        3.4.3
numpy                             1.21.2
...
````

- 仮想環境上で``pyinstaller``によって、``execute.py``をexe化する。
  - ``--onefile``：一つのexeファイルにまとめる
  - ``--clean``：exe化の度にクリーンする

````shell
(workdir) ~/workdir $ pyinstaller execute.py --onefile --clean
````

### 手順6
- ``sklearn``などを含む場合、``ModuleNotFoundError``が発生する。
  - ``ModuleNotFoundError: No module named ** to hidden-import in predict.spec``
- ModuleNotFoundErrorでひっかかるmoduleを``execute.spec``内の``hidden-import``に追加する。
  - ``sklearn``の場合、下記を追加
````spec
hidden-import = ['sklearn.utils._typedefs','sklearn.utils._heap', 'sklearn.utils._sorting', 'sklearn.utils._vector_sentinel', 'sklearn.neighbors._partition_nodes']
````

````shell
(workdir) ~/workdir $ pyinstaller execute.spec
````



## 開発履歴
- rev.0: 2022/07/28 イノベーションセンター 熊谷渉（Wataru.Kumagai@yokogawa.com）
