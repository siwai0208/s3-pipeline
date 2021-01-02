# s3 pipeline

## **About**

S3へのソースコード（Zip）のアップデートをトリガーに、EC2へのCodeDeployを実行するPipelineを作成

## **Inside Package**
* appspec.yml
* scripts

## **Dependency**
You have to execute [**VPC**](https://github.com/siwai0208/cloudformation/tree/main/vpc) and [**Security-Group**](https://github.com/siwai0208/cloudformation/tree/main/security-group) and [**S3**](https://github.com/siwai0208/cloudformation/tree/main/s3) and [**launch-templete**](https://github.com/siwai0208/cloudformation/tree/main/launch-templete)  stack before using this stack

## **How to use**

1. launch-templeteを使い、CodeDeployエージェントのインストールされたEC2を1台起動する（マネコン使用）
    EC2 ダッシュボード > 起動テンプレート > Codepipeline-template を選択
```
  アクション > テンプレートからインスタンスを起動<br>
  ネットワーク設定 > サブネット > public-subnet-1a を選択<br>
  テンプレートからインスタンスを起動
```

2. CodeDeployの作成

- デベロッパー用ツール > CodeDeploy > アプリケーション > アプリケーションの作成
```
  アプリケーション名: laravel-app<br>
  コンピューティングプラットフォーム: EC2/オンプレミス
```

- 続けて　デプロイグループの作成
```
  デプロイグループ名: laravel-app-dply-grp<br>
  サービスロール: CodeDeployServiceRole<br>
  環境設定: Amazon EC2 インスタンス<br>
  タググループ 1  キー:name 値:codepipeline<br>
  Load balancer: 無効<br>
  デプロイグループの作成
```

3. CodePipelineの作成
- デベロッパー用ツール > CodePipeline > パイプライン > パイプラインを作成する
```
  パイプライン名: s3-pipeline<br>
  サービスロール: 新しいサービスロール<br>
  アーティファクトストア: 作成済みのS3バケットを選択<br>
  ソースプロバイダー: Amazon S3<br>
  バケット: 作成済みのS3バケットを選択<br>
  S3 オブジェクトキー: laravel-app.zip<br>
  ビルドステージをスキップ<br>
  デプロイプロバイダー: AWS CodeDeploy<br>
  アプリケーション名: laravel-app<br>
  デプロイグループ: laravel-app-dply-grp<br>
  パイプラインを作成する<br>
```

- 作成完了時にpipelineが実施されるが、Zipファイルがなくエラーとなる

4. Zipファイルの作成

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
<br>Source 成功しました
<br>↓
<br>Deploy 成功しました

- EC2のパブリックIPにHTTPアクセスし、アプリが表示されることを確認する
