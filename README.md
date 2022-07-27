# PredictTool
需要予測ツール(exe版)の使い方

## dir構成
- ``input``: 入力データ用dir
    - サンプルデータ
        - 目的変数：``power [MWh]`` 東京電力パワーグリッド株式会社 東京エリア 電力実績[MWh] 【[URL][URL1]】
        - 説明変数：``temperature [deg]`` 気象庁 東京エリア 平均気温[℃] 【[URL][URL2]】
        - 期間：``2016/04/01 0:00 - 2018/03/31 23:00`` (17520個)
- ``output``: 出力データ用dir
- ``Preprocessing``: 前処理ツール用dir
- ``Predict``: 予測ツール用dir

## 実行手順
- 手順1: ``input``に入力データ（``input.csv``）を追加する（ファイル形式はサンプルデータを参照）
- 手順2: 前処理ツールの設定をする（``Preprocessing\preprocessing_config.ini``）
- 手順3: 前処理ツールを実行する（``Preprocessing\preprocessing.exe``をダブルクリック）
    - 結果: ``input``に加工済データ（``data_preprocessed.csv``）が出力・保存される
- 手順4: 予測ツールの設定をする（``Predict\predict_config.ini``）
- 手順5: 予測ツールを実行する（``Predict\predict.exe``をダブルクリック）
    - 結果: ``output``に学習データ（``tra.csv``）、検証データ（``val.csv``）、予測データ（``predict.csv``）、予測トレンド図（``predict.png``）が出力・保存される

## 機能
- Preprocessing
    - データ加工: 入力データを予測するための変数に変換・加工する
        - Timestampの分離: Timestamp列から新たな変数を追加する
            - 年／月／日／時刻／曜日を自動的に追加
        - Label-Encoding: カテゴリ列をlabel-encodingし、ラベル（整数値）に変換する
        - Onehot-Encoding: カテゴリ列をonehot-encodingし、バイナリ（0-1）に変換する
    - フィルタリング: 空白要素があれば、その行を自動でフィルタリングする
- Predict
    - 期間・変数設定: 入力データから予測に使用する部分を抜き出す
    - 学習・予測: 抜き出したデータと指定した予測手法を用いて、学習・予測する
    - 予測トレンド描画: 予測期間の実績値と予測値のトレンド図を描画する
- Error処理
    - 実行中でエラーが発生すると、``logフォルダに.log``をdumpする

## 前処理設定
- [DEFAULT]
    - ``SplitTimestamp``: Trueにすると、Timestamp系変数を生成する
        - timestamp形式は「yyyy/mm/dd HH:MM」 or 「yyyy-mm-dd HH:MM」しか対応していない
    - EncodeLabel: Trueにすると、カテゴリ列をlabel-encodingする
    - EncodeOnehot: Trueにすると、カテゴリ列をonehot-encodingする
        - MLRとPLSはonehot-encoding必須、RFではonehot-encoding不要
- [ENCODELABEL]
    - EncodeLabel=Falseに設定した場合、この設定は無視される
    - XList: label-encodingする列番号を指定（リスト形式で）
        - XList=[0,2]の場合、ファイル3列目と5列目を指定している（timestampと目的変数の列はカウントしない）
        - SplitTimestampで生成した列は、ここで指定する必要が無い（自動でencodeの対象となる）
            - つまり、XList=[]に設定した場合、Timestamp系の列のみ適用される
        - sklearnのLabelEncoderを使用しているため、詳細はsklearnのマニュアルを参照
- [ENCODEONEHOT]
    - EncodeOnehot=Falseに設定した場合、この設定は無視される
    - XList: one-hot-encodingする列番号を指定（リスト形式で）
        - XList=[0,2]の場合、ファイル3列目と5列目を指定している（timestampと目的変数の列はカウントしない）
        - SplitTimestampで生成した列は、ここで指定する必要が無い（自動でencodeの対象となる）
            - つまり、XList=[]に設定した場合、Timestamp系の列のみ適用される
        - pandasのget_dummiesを使用しているため、詳細はpandasのマニュアルを参照

## 予測設定
- [DEFAULT]
    - XList: 説明変数として使用する列番号を指定
        - XList=[0,2]の場合、ファイル3列目と5列目を指定している（timestampと目的変数の列はカウントしない）
    - TraPeriod, PrePeriod: 学習期間・予測期間を指定
        - timestampの形式: ``yyyy/mm/dd HH:MM`` or ``yyyy-mm-dd HH:MM``（これ以外の形式の場合は自分で変換しておく必要がある）
    - ModelingMode: 予測方法を指定（現在はMLR/PLS/RFのみ対応）
        - Python.scikit-learn(sklearn)のパッケージを使用しているため、詳細はsklearnのマニュアル[URL][URL3]を参照
        - MLR: sklearn.linear_model.LinearRegression
        - PLS: sklearn.cross_decomposition.PLSRegression
            - ハイパーパラメータはデフォルトで設定
        - Random Forest: sklearn.ensemble.RandomForestRegressor
            - ハイパーパラメータはデフォルトで設定

## 動作環境
- Windows10

## 開発履歴
- rev.0: 2022/06/17 熊谷渉

## その他
- 今後はこの機能構成をベースに、製品版エンジンのexeとして提供する予定


[URL1]: <https://www.tepco.co.jp/forecast/html/download-j.html>
[URL2]: <https://www.data.jma.go.jp/gmd/risk/obsdl/index.php>
[URL3]: <https://scikit-learn.org/stable/index.html>
