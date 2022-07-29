# Pythonコードをexeに変換する方法
Pythonコード（``.py``）をexeファイルに変換する方法
- 仮想環境でexe化する

## 使用ライブラリ
- 下記を利用した方法を説明する。
  - ``pyinstaller``: exe変換するための外部ライブラリ。
    - sklearnなどの外部ライブラリのパッケージはそのままexe化できない。exe化の前に、別途インポートが必要なサブモジュールを手動で設定しておく必要がある。
  - ``pipenv``: 仮想環境用の外部ライブラリ。``pip``（パッケージ管理）と``venv``（仮想環境構築）をまとめて使えるツール。
    - ローカル環境でexe化すると、ローカルのpython環境全てが対象となるため、exeファイルのサイズがかなり大きくなる。
    - exeファイルのサイズ削減のために、仮想環境上で必要なライブラリだけをインストールした後、exe化する。


## 手順
### 手順1：pipenvのインストール
- ローカル環境に``pipenv``をインストールする。
  - proxy設定は済ませておくこと

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

- ``execute.py``で``work_path``を取得するときは、下記のように``directly_flag``で切り替えられるようにしておく。
  - exeファイルで実行するときとpythonファイルで実行する場合で、``work_path``の認識がどうやら異なるらしい。
  - pythonファイルで実行する前: ``directly_flag = True``と設定
  - pythonファイルをexe化する前: ``directly_flag = False``と設定

```python 
[workdir\execute.py]
import os

directly_flag = True
if directly_flag:
    work_path = os.path.dirname( os.path.abspath(__file__) )
else:
    work_path = os.getcwd()
os.chdir(work_path)
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
- コマンドプロンプトを起動し、currentディレクトリを``workdir``まで移動させる。

````cmd
[local] $ cd workdir
````

- pipenvコマンドで、``workdir``内で仮想環境を構築（初期化）する。
  - ``workdir``内に仮想環境情報が記載されている``Pipfile``が作成される。

````cmd
[local] ~\workdir $ pipenv --python 3.9
````

- pipenvコマンドで、仮想環境内に``workdir\requiremenets.txt``に書いたパッケージをインストールする。
  - 通常のpip同様、proxy設定には注意。
  - 完了すると、``Pipfile.lock``が作成される。

````cmd
[local] ~\workdir $ pipenv install -r requirements.txt
````

### 手順5：仮想環境内への移動
- pipenvコマンドで、仮想環境内に移動する。

````cmd
[local] ~\workdir $ pipenv shell
(workdir) ~\workdir $ 
````

- ``pip list``コマンドで、仮想環境内にインストールされたパッケージを確認できる。

````shell
(workdir) ~\workdir $ pip list
Package                           Version
--------------------------------- ---------
matplotlib                        3.4.3
numpy                             1.21.2
...
````

### 手順6：仮想環境上でのexe化
- 仮想環境上で``pyinstaller``によって、下記のオプションをつけた状態で、``execute.py``をexe化する。
  - ``--onefile``：出力を一つのexeファイルにまとめる
  - ``--clean``：ビルド前に前回のキャッシュと出力ディレクトリを削除
  - 他にも、下記などがあるが、お好みで設定する。
    - ``--onedir``: 出力を1ディレクトリにまとめる
    - ``--noconsole``: コンソールを表示しない（ただし、exeを実行したとき、コンソールが表示されないと、実行しているかどうかが判別しにくくなる）

````shell
(workdir) ~\workdir $ pyinstaller execute.py --onefile --clean
````


- ``workdir``内に``dist``と``build``フォルダが作成され、``workdir\dist\execute.exe``が作成される。


#### サブモジュールのインポート問題の対処方法
- ただし、``sklearn``などの外部ライブラリを含む場合、exeを実行したときに``ModuleNotFoundError``が発生することがある。
  - いくつかの外部ライブラリを含んでexe化すると、サブモジュールが正しくインポートできないことがあるらしく、下記のエラーメッセージが出る。
  - ``ModuleNotFoundError: No module named ** to hidden-import in execute.spec``
- そこで、ModuleNotFoundErrorが出るサブモジュールをspecファイル``workdir\execute.spec``内の``hidden-import``に直接記述する。
  - ``sklearn``の場合、下記を``hidden-import``に追加する

````spec
hidden-import = ['sklearn.utils._typedefs','sklearn.utils._heap', 'sklearn.utils._sorting', 'sklearn.utils._vector_sentinel', 'sklearn.neighbors._partition_nodes']
````

- 次に、``pyinstaller``で、specファイル``workdir\execute.spec``を直接実行し、exeファイルを再作成（修正）する。

````shell
(workdir) ~\workdir $ pyinstaller execute.spec
````

### 手順7：exeファイルの実行
- ``workdir\dist\execute.exe``を``workdir``（``execute.py``と同じ階層）に移動させる。
  - データや自作ライブラリのインポート、データの出力・保存などは、``execute.py``の位置を基準に相対パスにしているはずなので、``execute.py``と同じ階層でexeを実行する
- ``workdir\execute.exe``をダブルクリックすると、実行される。
  - ただし、計算が正常に完了したのか、途中でエラーが起こったのか判別がつかないため、try~except構文でExceptionをキャッチし、logファイルをdumpするなど、工夫しておく
  - 外部ライブラリを含む場合、サブモジュールが正しくインポートできずに``ModuleNotFoundError``が発生することがある
    - ``workdir\execute.spec``内の``hidden-import``にそのサブモジュールを直接追記し、再度exe化し直す


## 開発履歴
- rev.0: 2022/07/28 イノベーションセンター 熊谷渉（Wataru.Kumagai@yokogawa.com）
