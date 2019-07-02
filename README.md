# sls-existing-apigw-lambda
作成済みのAPI GWとLambdaを連携させる

## 概要

[Serverless](https://serverless.com/framework/docs/providers/aws/events/apigateway/#share-api-gateway-and-api-resources) を読んで、すでにあるREST APIとリソースを指定することでCFnのコンフリクトが起きなくなると思ったので検証した。

## 検証内容

マネジメントコンソール上で API Gateway のAPIを作成し、以下の2パターンの状態で `sls deploy` を行い、エラーが起きるかどうかを確かめた
- `get-sls-message` リソースを作成するが、メソッドは作成しない
- 上に加えて、`GET` メソッドをMockとして作成する

## 結果

### リソースのみ作成した場合

- `sls deploy` に成功した
    - Lambda-proxy としてメソッドが作成された。
    - Lambdaのリソースポリシーも設定された　

### `GET` メソッドも登録した場合

- `sls deploy` に失敗した
- CFn のイベントを見ると「メソッドがすでにあるからダメです」と言われている

```
$ SLS_DEBUG='*' sls deploy -v
...

CloudFormation - CREATE_FAILED - AWS::ApiGateway::Method - ApiGatewayMethodGetDashslsDashmessageGet

...

  Serverless Error ---------------------------------------

  An error occurred: ApiGatewayMethodGetDashslsDashmessageGet - Method already exists for this resource (Service: AmazonApiGateway; Status Code: 409; Error Code: ConflictException; Request ID: 47b912a8-9cac-11e9-aaa5-a712f2bdd5c4).

```

## まとめ

- すでにメソッドがある状況では流石にどうしようもなさそう

## おまけ

### API Gateway 関係で使ったCLIコマンド

- REST API一覧を取得する。主にID取得が目的。

```
aws apigateway get-rest-apis
```

- REST API 内のリソースを取得する。

```
aws apigateway get-resources --rest-api-id {{ REST API ID }}
```

- REST API のリソースIDとパスの一覧を取得する

```
aws apigateway get-resources --rest-api-id {{ REST API ID}} --query "items[*].{id:id, path:path}" --output text | sort -k2
```