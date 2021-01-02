# s3 pipeline

## **About**

S3へのソースコード（Zip）のアップデートをトリガーに、EC2へのCodeDeployを実行するPipelineを作成

## **Inside Package**
* appspec.yml
* scripts

## **Dependency**
You have to execute [**launch-templete**](https://github.com/siwai0208/cloudformation/tree/main/launch-templete) stack and have your own S3bucket

## **How to use**

**1. launch-templeteを使い、CodeDeployエージェントのインストールされたEC2を1台起動する（マネコン使用）**

- EC2 ダッシュボード > 起動テンプレート > Codepipeline-template を選択
```
  アクション > テンプレートからインスタンスを起動
  ネットワーク設定 > サブネット > public-subnet-1a を選択
  テンプレートからインスタンスを起動
```

**2. CodeDeployの作成**

- デベロッパー用ツール > CodeDeploy > アプリケーション > アプリケーションの作成
```
  アプリケーション名: laravel-app
  コンピューティングプラットフォーム: EC2/オンプレミス
```

- 続けて　デプロイグループの作成
```
  デプロイグループ名: laravel-app-dply-grp
  サービスロール: CodeDeployServiceRole
  環境設定: Amazon EC2 インスタンス
  タググループ 1  キー:name 値:codepipeline
  Load balancer: 無効
  デプロイグループの作成
```

**3. CodePipelineの作成**
- デベロッパー用ツール > CodePipeline > パイプライン > パイプラインを作成する
```
  パイプライン名: s3-pipeline
  サービスロール: 新しいサービスロール
  アーティファクトストア: 作成済みのS3バケットを選択
  ソースプロバイダー: Amazon S3
  バケット: 作成済みのS3バケットを選択
  S3 オブジェクトキー: laravel-app.zip
  ビルドステージをスキップ
  デプロイプロバイダー: AWS CodeDeploy
  アプリケーション名: laravel-app
  デプロイグループ: laravel-app-dply-grp
  パイプラインを作成する
```

- 作成完了時にpipelineが実施されるが、Zipファイルがなくエラーとなる

**4. Zipファイルの作成**

- ソースコードを準備、今回は[Laravel-sample-app](https://github.com/siwai0208/food-app)を使用

- アプリのルート(/)配下に appspec.yml と scriptsディレクトリ を配置

```
    /
    ├── app
    ├── appspec.yml
    ...
    ├── scripts
    ...
```

- zipコマンドでルート配下をlaravel-app.zipファイルに保存する
```
  zip -r laravel-app.zip .
```

- s3バケットに作成したzipファイルをアップロードする
```
  $ aws s3 cp laravel-app.zip s3://S3バケット名/laravel-app.zip
```

- デベロッパー用ツール > CodePipeline > パイプライン > s3-pipeline
```
  Source 成功しました
  ↓
  Deploy 成功しました
```

- EC2のパブリックIPにHTTPアクセスし、アプリが表示されることを確認する
