---
title: "Cloudflare Birthday Week 2026 Day2"
emoji: "🎂"
type: "idea"
topics: ["cloudflare", "security", "tls", "waf", "ai"]
published: true
publication_name: "remotive"
---

Cloudflare Birthday Week 2026 の2日目です！
Day1 は Workers や Vite など開発者向けの発表が多かったですが、Day2 は認証局や耐量子暗号、WAF などセキュリティ周りの発表が中心です。

今日の領域は本当に素人なので誤りがあればコメントやGitHubなどでお伝えいただければなと思います！

Day1 の記事はこちらです。

https://zenn.dev/remotive/articles/cloudflare-birthday-week-2026-day1

今回も、公式の情報が欲しいかたは以下からどうぞ

https://www.cloudflare.com/birthday-week/

## Cloudflare が公開認証局に

Cloudflare が、公開証明書を発行する認証局（CA）になる方針を発表しました。

https://blog.cloudflare.com/cloudflare-certificate-authority/

Chrome・Apple・Microsoft・Mozilla のルートプログラムへの申請に加えて、GlobalSign が持つ、すでに広く信頼されているルートを取得する契約を結んだとのことです。

新しいルート証明書が世界中の端末に行き渡るには時間がかかりますし、更新が止まっている端末には届かないこともあります。既存のルートを取得することで、そうした古い端末にも対応しつつ、新しいルートの普及を進めるようです。

現時点ではまだ証明書の発行は行えませんが発行する証明書は無料で、取得・更新には ACME を使う予定とのことです。また、CA から証明書の更新時期を伝える ACME Renewal Information（ARI）に対応したクライアントを、発行の条件にするとのこと。

Universal SSL で証明書を配っていた Cloudflare が、今度は発行する側にもということで...Let's Encrypt などとあわせて、無料の証明書を取得できる選択肢が増えるのはありがたいことでしょう。

### Merkle Tree Certificates にも対応予定

この認証局では、耐量子暗号向けの Merkle Tree Certificates（MTC）にも対応する予定です。仕組みを説明する記事も別に公開されています。

https://blog.cloudflare.com/pq-ca-with-mtcs/

耐量子暗号の署名は従来のものより大きくなるため、そのまま証明書チェーンに使うと、TLS のハンドシェイクや Certificate Transparency のログに負担がかかりますが、それらを一定解決するようなもののようです。（このあたり詳しくないので詳しいかたにお任せしたい...）

標準的な MTC の発行も無料で、最初の本番発行は2027年第1四半期を目指しています。

## 自分のドメインの耐量子暗号対応を確認できるように

HTTP Traffic Analytics、Log Explorer、Logpush で、TLS の鍵交換に使われているアルゴリズムを確認できるようになりました。

https://blog.cloudflare.com/post-quantum-visibility/

これまでも TLS 1.3 などのバージョンは確認できましたが、今回わかるようになるのは、その接続で実際にどの鍵交換方式が使われたかです。

Dashboard の `Analytics > HTTP Traffic` では、訪問者からの HTTP リクエストを、接続に使われた鍵交換方式ごとに集計できます。`X25519MLKEM768` は、従来の X25519 と耐量子の ML-KEM を組み合わせた方式です。

![HTTP Traffic Analytics で、訪問者との接続に使われた鍵交換方式を確認する画面](/images/cloudflare-birthday-week-2026-day2/tls-key-exchange.webp =600x)
*鍵交換方式ごとの内訳。公式記事のテスト用ドメインの例です（[Cloudflare Blog](https://blog.cloudflare.com/post-quantum-visibility/) より引用）*

ログには、次のフィールドが追加されています。

- `ClientTLSKeyExchangeGroup`：訪問者と Cloudflare の間の鍵交換方式。Log Explorer と Logpush で確認できます。
- `OriginTLSKeyExchangeGroup`：Cloudflare とオリジンサーバーの間の鍵交換方式。こちらは Logpush で確認できます。

TLS 1.3 が有効で、訪問者側も `X25519MLKEM768` に対応していれば、Cloudflare は自動的にこの方式を使います。

自分のドメインに来ている通信が実際にどこまで対応しているのか、画面やログで追えるようになるのはちょっと面白いですね。

## 暗号の利用箇所を AI で調べる CryptoLabe

Cloudflare は2029年を目標に、プラットフォーム全体の耐量子暗号への移行を進めています。そのために社内で開発しているのが CryptoLabe というツールで、これが紹介されています。

https://blog.cloudflare.com/ai-driven-cryptography-discovery/

ソースコードや設定ファイル、依存関係などを AI で調べて、暗号がどこで、何のために使われているかを整理します。単に `RSA` などの文字列を探すだけだと、使われていないコードを拾ったり、ライブラリのデフォルト設定で使われる暗号を見落としたりする、というのが背景にあるようです。

まず利用箇所の候補を探し、その後に実行時の使われ方や関連リポジトリまで確認する、という2段階で調査します。根拠が足りないものは不明として残し、結果は担当のエンジニアが確認する流れです。

また、外部ライブラリや認証サービスの対応待ちなど、自分のチームだけでは移行できない理由も整理します。
実は同じような仕組みが別な移行などでも応用できるのでは...と思えてきますね...

### Cloudflare Developer Platform 上で構築されている

このツール自体の構成が紹介されていて、実際に Developer Platform 上で動いているようです。

https://blog.cloudflare.com/ai-driven-cryptography-discovery/#built-on-cloudflares-developer-platform

Workers と D1 で Dashboard や結果の管理を行い、Agents SDK の Durable Object でリポジトリごとの調査を管理、調査の各段階は Workflows で実行し、コードの確認には Sandbox、モデルの呼び出しには AI Gateway と Workers AI を使うというてんこ盛りですね。

![Workers、Durable Object、Workflows、Sandbox、Workers AI などを組み合わせた CryptoLabe の構成図](/images/cloudflare-birthday-week-2026-day2/cryptolabe-architecture.webp)
*CryptoLabe の構成（[Cloudflare Blog](https://blog.cloudflare.com/ai-driven-cryptography-discovery/) より引用）*

調査開始時のコミットに固定したコードを R2 に保存し、Sandbox ではそのスナップショットを読み取り専用のツールで調べるとのことです。長い調査の途中でコードが更新されても、見ている対象が変わらないようになっています。

CryptoLabe 本体は開発中の社内ツールで、現時点で顧客向けの提供はありません。今回公開されているのは、調査に使うプロンプトの一部です。

https://github.com/cloudflare/crypto-discovery-prompts/

暗号の移行という題材もそうですが、他にも何かしらの大規模移行が必要なケースなどで似たような手段が取れるかも...?ですね

## リクエストの形を学習する Application Profiles

WAF の機能追加である Application Profiles は、アプリケーションが受け取る想定のリクエストをプロファイルとして持ち、そこから外れたものを検出する機能です。実際の通信から学習するほか、OpenAPI のスキーマをアップロードすることもできます。

https://blog.cloudflare.com/application-profiles/

現在使える Schema Profile では、パスやクエリ、ヘッダー、Cookie、JSON や URL エンコードされたフォームの Body などを扱います。たとえば `product_id` が整数であると学習した場合に、文字列が送られてきたら違反として検出する、というイメージです。

既知の攻撃パターンに一致するかどうかに加えて、そのアプリケーションが受け取る想定のリクエストかどうかでも判断する、Positive Security の考え方です。API Security の Schema Learning・Schema Validation を、Web アプリケーションにも広げるものとして紹介されています。

![Application Profiles で学習した、ヘッダーやクエリパラメーターの型・値の範囲を確認する画面](/images/cloudflare-birthday-week-2026-day2/application-profile-schema.webp =600x)
*学習したスキーマを確認する画面。OpenAPI 形式で出力することもできます（[Cloudflare Blog](https://blog.cloudflare.com/application-profiles/) より引用）*


API Security を利用している場合はすでに使えますが、それ以外では、招待された Enterprise 顧客向けの Closed Beta です。

https://developers.cloudflare.com/waf/detections/application-profiles/

学習したスキーマを LLM で読んで、重要なフィールドや優先して守る操作を提案する機能も開発中とのことです。

### 学習・検知と、ブロックするルールの設定

Web Assets で対象の操作から `Learn profile` を選ぶと、過去7日間に 2xx を返したリクエストをもとに、週1回の学習が行われます。操作は HTTP メソッド・ホスト名・パスの組み合わせで、項目の学習には操作ごとに1,000件以上、値の範囲の学習には10,000件以上のリクエストが必要です。

条件を満たしてもすぐにプロファイルができるわけではなく、最初の学習結果が出るまで最大7日かかります。

プロファイルができると、実際の通信に対する検証が始まります。学習したスキーマへの違反は `cf.schema_validation.learned.violated` などのフィールドに入り、それを条件に Custom Rules でブロックする範囲を決めます。検証だけで自動的にブロックされるわけではありません。

アプリの更新や新しいクライアントによって、正常なリクエストの形が変わることもあります。まず Profile Analysis で検知結果や過去の通信への影響を確認してから、ブロックする範囲を決める流れです。

学習したプロファイルでは、必須パラメーターの欠落は検証されないため、未学習のパラメーターが増えただけでも違反にはならなりません。学習する場合には仕様にない入力をすべて拒否するものではないので、必須項目を指定して検証したい場合は、OpenAPI のスキーマをアップロードする方法が利用できます。

また、学習可能なプロファイルには制約がある（GraphQLが使えないなど）ので、注意が必要なこともありそうです。

https://developers.cloudflare.com/waf/detections/application-profiles/schema-profiles/


## AI を使って自社の WAF を検証した話

少し新機能などとは違いますが Cloudflare 自身の WAF に対して、LLM を使って攻撃のバリエーションを試した検証結果も公開されています。

https://blog.cloudflare.com/adaptive-ai-waf-testing/

許可を得た顧客のステージング環境で、ブロックされたリクエストの送り方やエンコードを変え、レスポンスを見て次の試行を決める、という検証を行っています。モデルにはアプリのソースコードや WAF ルールの中身を見せていないため、ある種のクリーンルーム的な検証になっているようです。

主な対象は XSS、SQL インジェクション、コマンドインジェクション、SSRF、パストラバーサル・LFI、Log4j の6カテゴリです。別枠のログインジェクションを含め、45のシナリオで試しています。

全体で1,107回の試行があり、形式が壊れたリクエストや無害な入力、重複などを人手で整理した後の集計は607件で、内訳は WAF がブロックした558件と、追加調査の対象になった49件です。この49件のうち48件が、コマンドインジェクションと SSRF に関するものでした。

ここでいう49件は、アプリケーションへの攻撃が49回成功したという意味ではありません。WAF を通過したリクエストについて、実際に悪意のある内容か、再現できるか、WAF で扱うべき問題かを確認して残った調査対象です。

検証の成果は、7月21日の Managed Ruleset 更新での SSRF 検知追加などにもつながっています。生成したリクエストを試すところから、再現性や誤検知への影響を確認してルールに反映するところまで、具体的な流れが紹介されていて面白いなと思いました。

## アプリケーションのセキュリティを継続的に改善する仕組み

最後に、今回の発表と既存の製品をどう組み合わせ、どう継続的に改善していくのか、という方針も紹介されています。

https://blog.cloudflare.com/ai-era-framework/

大きくは、次の4つの段階をつなげていくというものです。

1. リスクを見つけて、対応の優先順位を決める
2. 人や Agent が何にアクセスし、何をしてよいか管理する
3. 実際に動いているアプリケーションを保護する
4. 調査・対応の結果を、次の検知や防御に反映する

たとえば、コードから見つかった脆弱性が実際の通信から到達できる場所にあるかを調べ、対応の優先順位や WAF の防御につなげます。さらに、その後の調査で得た情報を、次のルールや検知の改善にも使うという考え方です。

指定した URL を LLM の Agent が定期的に検証する Adaptive Security や、複数のデータから不審な動きを調べて対策を提案する仕組みも開発中です。後者は Managed Defense チームと検証しており、提案した対策は人間の承認を経て適用する形が説明されています。

## おわりに

Day1 からかなり雰囲気が変わって、セキュリティの話が多い1日でした。中でも、Cloudflare が認証局になるという発表はかなりインパクトが強いですね...

Qualys SSL Labs の SSL Server Test で何に対応してるかテストしたことのある人も多いんじゃないかと思うので、Traficの鍵交換に使われてるアルゴリズムが見えるのはちょっと楽しいかもしれませんね（？）

:::message
この記事は AI の支援を得て情報収集・執筆・校正を行っています。
:::
