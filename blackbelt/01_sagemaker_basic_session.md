## SageMaker BasicSession

これまでの課題

- 開発・学習
  - 環境構築大変
  - 複数の学習ジョブを並列で実行するのが大変
  - etc
- 推論
  - 推論用の API サーバー構築とメンテが大変
  - エッジデバイスへのデプロイが大変
  - バッチ推論の仕組みを構築するのが大変

要するに、データサイエンティストが機械学習の仕組みを作って、エンジニアがそれを API サーバーにデプロイしたりするのに対して、大変。

これらを解決するための SageMaker があるらしい

## 概要メモ

- Jupyter NoteBook をオンライン上で実施できるようだ
- テンプレートがたくさんある
  - 20219 年のブラックベルトではライブラリは Chainer で説明されていた
    - ビルドインアルゴリズム・・・データだけあればすぐできる
    - Tensoflow とか Chainer とか PyTorch とか・・・スクリプトとデータがあればすぐできる
    - 独自アルゴリズム・・・自分で ECR のイメージ作ったりとか必要
  - Pytorch とか
  - この辺りのテンプレがたくさんあるので、環境構築の必要がない
- SageMaker のコンポーネント
  - ラベリング
    - 機械学習のためのインプットデータ作成を支援。ウェブベースのツールを提供。画像や文章などに対して効率的にラベル付を行える
  - 開発
    - 学習するためのコードの記述・入力データの加工整形など。Jupyter NoteBook やその他ライブラリがインストール済みのインスタンスを提供
  - 学習
    - API を叩くと学習用インスタンスが立ち上がり、学習ジョブを実行。複数ジョブの同時実行や、複数インスタンスでの分散学習、ハイパーパラメータチューニングに対応
  - モデル変換
    - 学習済みのモデルに対して実行環境に最適化された形でのモデル変換機能を提供。EC2 インスタンスやエッジデバイスに最適化された形のモデルを作成できる
    - ハイパーパラメーターの調整とかも自動ですることが可能
  - 推論
    - API を叩くと指定したモデルを読み込んでオートスケーリングや AB テストに対応したエンドポイントが作背できる
    - 大量データをバッチで推論する処理もサポート

## SageMaker の学習・推論

### 開発

- ノートブック上で前処理やプロトタイピングも簡単に利用可能
- git リポジトリを SageMaker に設定しておけばすぐにコードを使うことも可能
- 環境変数の設定や・コンテナのライフサイクルとかも設定できる

### 学習

学習のアーキテクチャ

![学習イメージ](./images/learning.png)

- 分散処理もできるけど Tensorflow などの Python 処理は自分で書く必要がある
- `estimator.fit(wait=False)`とするとジョブの終了を待たない
- Estimater の初期化時に　 hyperparameters で引き渡すパラメータでハイパーパラメーターを自動で調整してくれる
- ローカルモードでテストをできる（いちいちインスタンスを立ち上げずにさくっっとテスト)
- アルゴリズム・ハイパーパラメーターの設定、タグなどで合致するデータを検索できるサーチサービスもあり

### 推論

推論のアーキテクチャ
![学習イメージ](./images/suiron.png)

- AB テストの機能もあり
- 複数のもでるそぞれぞれに、インスタンスタイプやインスタンス数とか設定可能
- 推論パイプライン
  - 複数の推論エンドポイントを一連おパイプラインとして定義可能
  - 前処理用のコンテナ・分類用のコンテナ・後処理用のコンテナというふうにパイプラインを組むことができる
  - ElasticInference CPU のみの EC2 に GPU のアクセラレーター処理を後からアタッチできたりする

### SageMaker の活用方法

- 開発・学習・推論のいずれかだけ利用するとか、利用しないとかもちろん可能
  - 本番は
    StepFunctions のワークフローで Glue や SageMaker について lamdba を使わずに利用することが可能
