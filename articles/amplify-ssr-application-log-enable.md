---
title: "Amplify Hosting v2 の SSR アプリケーションログを後から有効化する"
emoji: "🔼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Amplify", "AWS", "Web"]
published: true
---

Amplify Hosting v2 で SvelteKit を使った SSR アプリケーションをホスティングしているときに、プロジェクト作成のタイミングで「SSR アプリケーションログを有効にする」のチェックを入れていなかった関係でアプリケーションのログが一切CloudWatch Logsに記録されて居らず、500エラーが発生した場合などに原因特定が出来なくなっていました

SSR アプリケーションログを後から有効化する方法を調べてみましたがどこにも情報がなく、唯一出てきたドキュメントはこれだけでした
https://docs.aws.amazon.com/amplify/latest/userguide/get-started-sveltekit.html

どこにも設定メニューはなく、aws-cli でも設定できないようで困り果てて居ましたが、なんとか解決できたので他にも困った方が居たらと思い、残しておきます

## 手順

### 前提

- Amplify Hosting v2 で SvelteKit を使った SSR アプリケーションをホスティングしている
- aws-cli がインストールされていて、Hosting先にログインしている
- ログインユーザに iam関係の権限があること

### 必要な情報の収集
- Amplify の AppId
  - `aws amplify list-apps` で取得できる
  - 今回は `d123456789` とします
- AWS Region
  - 今回は `ap-northeast-1` とします
- AWS AccountId
  - `aws sts get-caller-identity` で取得できる
  - 今回は `123456789012` とします

### IAM Role の作成
Principal を `amplify.amazonaws.com` にして、Roleを作成します

```shell
$ aws iam create-role \
      --role-name aws-amplify-SSRComputeResourceLoggingWriteRole \
      --assume-role-policy-document \
      '{
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Principal": {
                  "Service": "amplify.amazonaws.com"
              },
              "Action": "sts:AssumeRole"
          }
      ]
  }'
```

作成に完了すると以下のようなjsonが帰ってきます

```json
{
    "Role": {
        "Path": "/",
        "RoleName": "aws-amplify-SSRComputeResourceLoggingWriteRole",
        "RoleId": "ABCDEFGHIJKLMNOPQ",
        "Arn": "arn:aws:iam::123456789012:role/aws-amplify-SSRComputeResourceLoggingWriteRole",
        "CreateDate": "2025-04-16T07:57:00+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "amplify.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}
```

### Inline Policy をアタッチ

次に、作成したRoleにInline Policyをアタッチします

```shell
$ aws iam put-role-policy \
      --role-name aws-amplify-SSRComputeResourceLoggingWriteRole \
      --policy-name AmplifySSRComputeResourceLoggingWriteAccess \
      --policy-document \
      '{
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowLoggroupWrite",
              "Effect": "Allow",
              "Action": [
                  "logs:CreateLogStream",
                  "logs:CreateLogGroup",
                  "logs:DescribeLogGroups",
                  "logs:PutLogEvents"
              ],
              "Resource": [
                  "arn:aws:logs:ap-northeast-1:123456789012:log-group:/aws/amplify/*:*"
              ]
          }
      ]
  }'
```

:::details カスタマー管理ポリシーを利用する場合

ポリシーの作成
```shell
$ aws iam create-policy \
      --policy-name AmplifySSRComputeResourceLoggingWriteAccess \
      --policy-document \
      '{
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "AllowLoggroupWrite",
              "Effect": "Allow",
              "Action": [
                  "logs:CreateLogStream",
                  "logs:CreateLogGroup",
                  "logs:DescribeLogGroups",
                  "logs:PutLogEvents"
              ],
              "Resource": [
                  "arn:aws:logs:ap-northeast-1:123456789012:log-group:/aws/amplify/*:*"
              ]
          }
      ]
  }'
```

次に、作成したPolicyをRoleにアタッチします
```shell
$ aws iam attach-role-policy \
  --role-name aws-amplify-SSRComputeResourceLoggingWriteRole \
  --policy-arn <Policy ARN>
```

:::

### Amplify App の更新
次に、Amplify Appのサービス IAM ロールに、作成したRoleを設定します。

```shell

$ aws amplify update-app \
      --app-id d123456789 \
      --service-role-arn arn:aws:iam::123456789012:role/aws-amplify-SSRComputeResourceLoggingWriteRole
```

その後、再度デプロイすることで、ログがCloudWatch Logsに出力されるようになります

CloudWatch Logsのロググループは `/aws/amplify/<app-id>` で作成されます。
（なので、専用にroleを作成するのであれば Resource の指定を `arn:aws:logs:ap-northeast-1:123456789012:log-group:/aws/amplify/<app-id>/*:*` にしても問題無いかと思われます）

#### 余談
これで解決することから、アプリケーション作成時の「SSR アプリケーションログを有効にする」のチェックボックスはこのポリシーを作成してアタッチするだけで、Backendのリソースが存在する状態であればずっとCloudWatch Logsに出力しようとしてはいるのではないかと思われます。 （AWS の詳しい人に聞いてみないとわからないですが）
今回はサービス IAM ロールにアタッチすることで、CloudWatch Logsに出力されるようになりましたが、もしかしたら、Compute ResourceのIAM Roleにアタッチしても出力されるかもしれません。 （試していないのでわからないですが）

### 参考
- https://docs.aws.amazon.com/amplify/latest/userguide/cloudwatch-logs-role.html
- https://repost.aws/questions/QUhUSDLVAbTD6zHwbcb5Xtog/how-can-i-send-logs-to-my-amplify-ssr-nextjs-app-to-cloudwacth
- https://github.com/aws-amplify/amplify-hosting/issues/3964