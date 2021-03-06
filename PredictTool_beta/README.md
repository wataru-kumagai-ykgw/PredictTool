# PredictTool_beta
需要予測ツールbeta版の使い方
- Pythonの予測手法を内蔵した予測ツール。exeで提供されるため、環境構築やプログラミングは不要。

## dir構成
- ``input``: 入力データ用dir
    - ``input.csv``: 入力データ（サンプル、東京の電力需要予測用ファイル一式）
        - 目的変数：``power [MWh]`` 東京電力パワーグリッド株式会社 東京エリア 電力実績[MWh] 【[URL][URL1]】
        - 説明変数：``temperature [deg]`` 気象庁 東京エリア 平均気温[℃] 【[URL][URL2]】
        - 期間：``2016/04/01 0:00 - 2018/03/31 23:00`` (17520個)
    - ``data_preprocessed.csv``: 前処理によって加工したデータ（サンプルだが、実行する度に上書きされる）
- ``output``: 出力データ用dir
    - 予測結果ファイル一式（サンプルだが、実行する度に上書きされる）
- ``Preprocessing``: 前処理ツール用dir
    - ``preprocessing.exe``: 前処理ツールのexe。
    - ``preprocessing_config.ini``: 前処理ツールの設定ファイル。
- ``Predict``: 予測ツール用dir
    - ``predict.exe``: 予測ツールのexe。
    - ``predict_config.ini``: 予測ツールの設定ファイル。

## 実行手順
- 手順1: ``input``に入力データ``input.csv``を追加する（ファイル形式はサンプルデータを参照）
- 手順2: ``Preprocessing\preprocessing_config.ini``から、前処理ツールの設定をする
- 手順3: ``Preprocessing\preprocessing.exe``をダブルクリックし、前処理ツールを実行する
    - 結果: ``input``に加工済データ``data_preprocessed.csv``が出力・保存される
- 手順4: ``Predict\predict_config.ini``から、予測ツールの設定をする
- 手順5: ``Predict\predict.exe``をダブルクリックし、予測ツールを実行する
    - 結果: ``output``に予測結果ファイル一式が出力・保存される

## 機能
- ``preprocessing.exe``：``input\input.csv``に対して、下記機能を実行する。
    - フィルタリング: 空白要素があれば、その行を自動でフィルタリングする。
    - データ加工: 入力データを予測するための変数に変換・加工する。
        - Timestampの分離: Timestamp列から新たな変数を追加する
            - 年／月／日／時刻／曜日／休日用変数を自動的に追加する
            - ただし、休日用変数は__日本のカレンダーから祝日のみを抽出__している。
                - 土曜／日曜は曜日用変数に含まれるため、除外している
                - 日本国外データの場合は上記の祝日を使わず、各自で``input\input.csv``に休日用変数を追加する
                - 例えば12月31日などは祝日として扱われないため、厳密な休日用変数ではない。必要に応じて、各自で``input\data_preprocessed.csv``の休日用変数を修正する
        - Label-Encoding: カテゴリ列をlabel-encodingし、ラベル（整数値）に変換する
            - example (Table 1): [Wind_Direction] --> [Wind_Direction_label]
        - Onehot-Encoding: カテゴリ列をonehot-encodingし、バイナリ（0-1）に変換する
            - example (Table 2): [Wind_Direction] --> [Wind_Direction_East,Wind_Direction_West,Wind_Direction_South,Wind_Direction_North]
- ``predict.exe``：``input\data_preprocessed.csv``に対して、下記機能を実行する。
    - 期間・変数設定: 入力データから学習・予測に使用する部分を抜き出す。
    - 学習・予測: 指定したデータと指定した予測手法を用いて、学習・予測する。
    - 予測トレンド描画: 予測期間の実績値と予測値のトレンド図を描画する。
        - ただし、予測期間に応じて図をリサイズしないため、予測期間が長すぎると、見えづらくなる。
- Error処理
    - exe実行中でエラーが発生すると、exeと同じフォルダに``log``フォルダを作成し、``log.txt``をdumpする。
    - エラー内容が不明な場合、入力データ、設定ファイル、``log.txt``などのファイル一式を担当者に送って問い合わせる。

Table 1: label-encoding
| Wind_Direction | Wind_Direction_label |
| -------------- | -------------------- |
| East  |  1  |
| West  |  2  |
| South |  3  |
| North |  4  |
| East  |  1  |
| North |  4  |

Table 2: onehot-encoding
| Wind_Direction | Wind_Direction_East | Wind_Direction_West | Wind_Direction_South | Wind_Direction_North |
| -------------- | ------------------- | ------------------- | -------------------- | -------------------- |
| East  |  1  |  0  |  0  |  0  |
| West  |  0  |  1  |  0  |  0  |
| South |  0  |  0  |  1  |  0  |
| North |  0  |  0  |  0  |  1  |
| East  |  1  |  0  |  0  |  0  |
| North |  0  |  0  |  0  |  1  |


## input/outputのファイル形式
- 入力データ以外は、実行の度に同じファイル名で上書きされる。上書きされたくない場合は``output``フォルダ毎、リネームしておく。
- ``input\input.csv``: 入力データ。
    - 1行目: 各列の名称。``timestamp``以外の列名は、空でなければ何を入力しても結果には影響しない。 
    - 2行目以降: 数値データ
    - 1列目: timestamp。``yyyy/mm/dd HH:MM`` or ``yyyy-mm-dd HH:MM``の形式以外は受け付けない（これ以外の形式の場合は自分で変換しておく必要がある）
    - 2列目: 目的変数
    - 3列目以降: 説明変数
- ``input\data_preprocessed.csv``: 加工済データ。前処理ツールの結果、生成される。
    - 1列目: timestamp（フィルタリングによって削除された行は、復元されない）
    - 2列目: 目的変数
    - 3列目以降: 説明変数（データ加工によって新規に生成した変数が追加されている）
- ``output\tra.csv``: 学習期間に該当するデータ。予測ツールの結果、生成される。
    - 各列: ``data_preprocessed.csv``から指定した目的変数・説明変数
    - 各行: ``data_preprocessed.csv``から指定した学習期間
- ``output\val.csv``: 予測期間に該当するデータ。予測ツールの結果、生成される。
    - 各列: ``data_preprocessed.csv``から指定した目的変数・説明変数
    - 各行: ``data_preprocessed.csv``から指定した予測期間
- ``output\predict.csv``: 学習済モデルに予測期間の説明変数を入力することで予測したデータ。予測ツールの結果、生成される。
    - 各行: ``data_preprocessed.csv``から指定した予測期間
    - ``XX_pre``: 予測値
    - ``XX_act``: 実績値
- ``output\predict.png``: ``predict.csv``のデータをトレンドにした描画した図。予測ツールの結果、生成される。
    - 橙線は予測値
    - 青線は実績値


## 前処理設定
- ``Preprocessing\preprocessing_config.ini``の詳細。``input\input.csv``に対して前処理する条件を設定する。
- ``[DEFAULT]``
    - ``SplitTimestamp``: ``True``にすると、Timestamp系変数を生成する。``False``にすると、Timestamp系変数を生成しない。
        - 現在、timestamp形式は``yyyy/mm/dd HH:MM`` or ``yyyy-mm-dd HH:MM``しか対応していない
        - 現在、日本の祝日抽出は、Python外部ライブラリ``jpholiday``を使用しているため、詳細は``jpholiday``のマニュアル[URL][URL3]を参照
    - ``EncodeLabel``: ``True``にすると、カテゴリ列をlabel-encodingする。``False``にすると、label-encodingしない。
    - ``EncodeOnehot``: ``True``にすると、カテゴリ列をonehot-encodingする。``False``にすると、onehot-encodingしない。
        - ``MLR``と``PLS``を使う場合、onehot-encoding必須、``RF``を使う場合、onehot-encoding不要
- ``[ENCODELABEL]``
    - ``EncodeLabel = False``に設定した場合、この設定は無視される
    - ``XList``: label-encodingする列番号を指定（リスト形式で）
        - ``XList=[0,2]``の場合、``input/input.csv``の3列目と5列目を指定している（``timestamp``と目的変数の列はカウントしない）
        - ``SplitTimestamp``で生成した列は、ここで指定する必要が無い（自動でencodeの対象となる）
            - つまり、``XList=[]``に設定した場合、Timestamp系の列のみ適用される
        - Python外部ライブラリ``scikit-learn(sklearn)``のパッケージ``LabelEncoder``を使用しているため、詳細は``sklearn``のマニュアル[URL][URL4]を参照
- ``[ENCODEONEHOT]``
    - ``EncodeOnehot = False``に設定した場合、この設定は無視される
    - ``XList``: one-hot-encodingする列番号を指定（リスト形式で）
        - ``XList=[0,2]``の場合、ファイル3列目と5列目を指定している（``timestamp``と目的変数の列はカウントしない）
        - ``SplitTimestamp``で生成した列は、ここで指定する必要が無い（自動でencodeの対象となる）
            - つまり、``XList=[]``に設定した場合、Timestamp系の列のみ適用される
        - Python外部ライブラリ``pandas``のパッケージ``get_dummies``を使用しているため、詳細は``pandas``のマニュアル[URL][URL5]を参照

## 予測設定
- ``Predict\predict_config.ini``の詳細。``input\data_preprocessed.csv``に対して学習・予測する条件を設定する。
- ``[DEFAULT]``
    - ``XList``: 説明変数として使用する列番号を指定
        - ``XList=[0,2]``の場合、``input\data_preprocessed.csv``の3列目と5列目を指定している（``timestamp``と目的変数の列はカウントしない）
        - ``input\data_preprocessed.csv``を直接確認しながら、列番号を指定すれば良い
    - ``TraPeriod``, ``PrePeriod``: 学習期間・予測期間を指定
        - ``yyyy/mm/dd HH:MM`` or ``yyyy-mm-dd HH:MM``の形式のみ受け付ける（``input\data_preprocessed.csv``の``timestamp``の形式に合わせれば良い）
        - ``input\data_preprocessed.csv``に含まれていない学習期間・予測期間を設定すると、エラーとなる。
            - 目的変数の実績値が欠落しているが予測したい期間がある場合、前処理の前に、``input\input.csv``内の該当する期間の目的変数に0を入れておく。（空だとフィルタリングで除外されるため）
    - ``ModelingMode``: 予測方法を指定
        - ``MLR``/``PLS``/``RF``の3種類から選ぶ。推奨設定（default）は``ModelingMode = RF``。
            - ``MLR``: multi linear regression（線形重回帰）。線形回帰モデル。
            - ``PLS``: partial least squares regression（偏最小二乗回帰）。線形回帰モデル。
            - ``RF``: Random Forest。非線形回帰モデル。
        - Python外部ライブラリ``scikit-learn(sklearn)``を使用しているため、詳細は``sklearn``のマニュアル[URL][URL4]を参照
            - MLR: ``sklearn.linear_model.LinearRegression``を使用。
            - PLS: ``sklearn.cross_decomposition.PLSRegression``を使用。
            - Random Forest: ``sklearn.ensemble.RandomForestRegressor``を使用。
- ``[PLS]``: PLS用のハイパーパラメータ。
    - ``NumLV``: 潜在変数の数。推奨設定（default）は``NumLV = 2``。
    - これ以外のパラメータは外部ライブラリのデフォルトが適用される。
- ``[RANDOMFOREST]``: Random Forest用のハイパーパラメータ。
    - ``NumTree``: 木の数。推奨設定（default）は``NumTree = 100``。
    - ``MaxDepth``: 木の深さ。推奨設定（default）は``MaxDepth = None``。
    - ``MinSamplesSplit``: 葉内のサンプル数。推奨設定（default）は``MinSamplesSplit = 2``。
    - これ以外のパラメータは外部ライブラリのデフォルトが適用される。

## 動作環境
- Windows10

## 開発履歴
- rev.0: 2022/06/17 イノベーションセンター 熊谷渉（Wataru.Kumagai@yokogawa.com）

## その他
- 今後はこの機能構成をベースに、製品版エンジンのexeとして提供する予定


[URL1]: <https://www.tepco.co.jp/forecast/html/download-j.html>
[URL2]: <https://www.data.jma.go.jp/gmd/risk/obsdl/index.php>
[URL3]: <https://pypi.org/project/jpholiday/>
[URL4]: <https://scikit-learn.org/stable/index.html>
[URL5]: <https://pandas.pydata.org/>
