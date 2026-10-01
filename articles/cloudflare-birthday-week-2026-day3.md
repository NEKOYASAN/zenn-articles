---
title: "Cloudflare Birthday Week 2026 Day3"
emoji: "🎂"
type: "idea"
topics: ["cloudflare", "cloudflareworkers", "ai", "containers", "mcp"]
published: true
publication_name: "remotive"
---

Cloudflare Birthday Week 2026 の3日目です！
Day2 はセキュリティ周りが中心でしたが、Day3 は Containers や Workers のエラー監視、AI Gateway など、また開発者として気になる発表が増えています。

AI Agent がサービスやコンテンツを使ったときに、どうお金を払うかという話も出てきました。

Day2 の記事はこちらです。

https://zenn.dev/remotive/articles/cloudflare-birthday-week-2026-day2

公式の発表一覧は以下からどうぞ

https://www.cloudflare.com/birthday-week/

## Containers の起動が高速化、実行時に構成を選べるように

Cloudflare Containers に、`durable_object` という新しいスケジューリングポリシーが Public Beta として追加されました。起動の高速化に加えて、コンテナを起動するタイミングでイメージやインスタンスタイプを選べます。

https://blog.cloudflare.com/faster-agent-sandboxes/

これまではイメージとリソースの組み合わせをデプロイ時に決めていましたが、設定に用意しておいた Node.js 用・Python 用などのイメージを、処理の内容に応じて選べるようになります。ビルドには大きめのインスタンスを使う、といった判断もコードで書けます。

起動時間については、100個の Sandbox を同時に起動する ComputeSDK のベンチマークで、操作可能になるまでの中央値が 4.049s から 648ms になったとのことです。約6.2倍の高速化で、Agent が作業を始める前の待ち時間がかなり減りそうです。

Cloudflare Workers Tech Talks in Tokyo #8 で sh1ma さんが AWS Lambda の MicroVM との比較と高速化手法を試されていたので、それらとの兼ね合いや組み合わせなどでさらに短縮する余地があるのかは気になりますね...

https://www.youtube.com/live/pDdK_gHt8fQ?si=ApKm03anfufKQCbP&t=1483

### 作業環境をスナップショットで保存できる

ファイルシステムのスナップショットも Public Beta になりました。リポジトリやインストール済みの依存関係、作業途中のファイルなどを保存して、次にコンテナを起動するときに復元できます。

https://developers.cloudflare.com/containers/guides/snapshots/

同じスナップショットから複数のコンテナを起動することもできるので、環境を揃えたままモデルやプロンプトを変えて評価する、といった用途にも使えます。

保存されるのはファイルシステムで、メモリや実行中のプロセスは含まれません。また、スナップショットは作成時のイメージのバージョンに紐づくので、別のイメージへそのまま持っていくことはできないようです。

毎回リポジトリを clone して依存関係を入れ直すところまで待つ必要がなくなるのは、Agent の作業環境を作るうえでかなり嬉しいですね。

### `Container` クラスから `ctx.container` へ

もともと Containers は `Container` クラスの裏に `DurableObject` を隠していましたが、今回の新機能を使う場合は、`Container` クラスの代わりに `DurableObject` を継承し、`this.ctx.container` からコンテナの起動やコマンド実行を行う必要があります。

たとえば、Agent の会話履歴や作業の進み具合は Durable Object で管理し、ビルドやテストが必要なときだけコンテナを起動する、といった構成が行いやすくなります。Agent の状態を管理する部分と、Linux 上で実際に作業する部分を分けて書くイメージですね。

![Durable Object がコンテナの作業環境、外部通信、評価処理を管理する3つの構成例](/images/cloudflare-birthday-week-2026-day3/containers-durable-object.webp)
*Durable Object と Container を組み合わせた Agent の構成例（[Cloudflare Blog](https://blog.cloudflare.com/faster-agent-sandboxes/) より引用）*

既存の `Container` クラスと従来の `Sandbox` クラスの保守は、2026年12月31日までとのことです。その日を過ぎて既存のデプロイが停止するわけではありませんが、クラスの更新は終了するため、移行が推奨されています。

Sandbox SDK 1.0 も、継承して使う基底クラスから、自分の Durable Object の中で使うユーティリティ集に変わります。割と書き方が大きく変わるので、migration guideを要チェックですね

https://developers.cloudflare.com/containers/guides/migrate-to-durable-object-container-api/

## Workers のエラーをまとめて Agent に渡す Issues

Workers にエラー監視機能の Issues が追加され、Open Beta で利用できるようになりました。未処理の例外や 5xx レスポンス、エラーログなどを検出し、関連するエラーを1つの Issue にまとめます。

https://blog.cloudflare.com/real-time-issue-detection/

同じバグで何千回もエラーが出ていても、1件ずつログを追うのではなく、いつから起きているか、何回発生しているか、増えているかを確認できます。詳細画面では、スタックトレースや前後のログ・トレース、Worker のバージョンなども一緒に見られます。

![Workers の Issue 詳細画面で、エラー、実行の流れ、発生件数、Worker の情報を確認している例](/images/cloudflare-birthday-week-2026-day3/workers-issue-details.webp)
*Issue にまとめられたエラーと調査用の情報（[Cloudflare Blog](https://blog.cloudflare.com/real-time-issue-detection/) より引用）*

Workers のランタイムに組み込まれているため、SDK やアプリケーションのラッパーを追加する必要はありません。

Wrangler 4.134.0 以降では、設定の `observability.issues.enabled` を `true` にし、再デプロイします。[Day1 で発表された `cf` を使う場合](https://zenn.dev/remotive/articles/cloudflare-birthday-week-2026-day1#cloudflare-%E5%85%A8%E4%BD%93%E3%82%92%E6%93%8D%E4%BD%9C%E3%81%99%E3%82%8B%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89-cf)は、`cloudflare.config.ts` の `worker.observability.issues.enabled` で設定できます。

すべての Workers アカウントが対象で、Beta 期間中は無料です。有効化した後の通信から検出が始まるため、過去のエラーが遡ってまとまるわけではありません。

https://developers.cloudflare.com/workers/observability/issues/

最近若干 Observability が強化されていて、Trace データなどが取れるようになってきたところで、エラーに関連する Log や Trace をまとめて確認できる機能が出てくるのは素直に嬉しいです。（[Workers Logs は構造化ログにも対応している](https://developers.cloudflare.com/workers/observability/logs/workers-logs/#logging-structured-json-objects)ので、調査に必要な情報を一緒に出しておきたいですね）

### 条件を指定してAIや各種支援ツールと連携する Automations

Automations を設定すると、一定回数の発生や、しばらく落ち着いていた問題の再発をきっかけに、Issue の情報を送れます。Claude Code、Cursor、Devin との連携や、汎用の Webhook、チャット・障害対応ツールへの通知が用意されています。

Agent にエラーと周辺情報を渡して、調査や修正の PR 作成につなげる流れです。追加のログを調べさせたい場合は Cloudflare MCP も接続でき、修正のレビューやデプロイは人間が行います。

Cloudflare 自身も Workflows の運用に使っていて、移行処理のリトライループや、削除が完了しない問題を見つけた例が紹介されています。ログを集めて Agent に渡すところがまとまるのは、普段の開発でも助かりそうです。

## AI Gateway がモデルを選ぶ Auto Router

AI Gateway に、リクエストの内容に応じてモデルを選ぶ Auto Router が Public Beta として追加されました。モデル名に `cloudflare/auto` を指定すると利用できます。

https://blog.cloudflare.com/auto-router/

まず入力形式や認証情報、予算などの条件から利用できるモデルを絞り、会話の内容からタスクの種類や難しさを分類します。そのうえで、期待する回答の品質とコストをもとにモデルを選ぶ仕組みです。分類用のモデル自体は Workers AI で動いています。

簡単な整形や要約なら軽いモデル、複雑なコード修正ならより強いモデル、といった使い分けを Gateway 側に任せるイメージです。

Cloudflare 社内の OpenCode での初期利用では、フロンティアモデルだけを使う場合と比べて最大30%のコスト削減が見られたとのことでした。

OpenRouter の [Auto Router](https://openrouter.ai/docs/guides/routing/routers/auto-router) と同じような動きになるんでしょうか... Cloudflare で使うならば AI Gateway の方が扱いやすいのでありがたいですね！

### 長い会話ではキャッシュも考慮する

モデルを切り替えるたびに長いコンテキストを読み直すと、モデルの単価が安くても、全体では高くなることがあります。

`cf-aig-session-id` を付けると、ユーザーの1回の入力から続くツール呼び出しなどを同じモデルで処理し、次のターンで切り替えるときも、キャッシュを失うコストを考慮します。セッション ID を付けない場合は、リクエストごとにモデルを選びます。

選ばれたモデルは `cf-aig-routed-model` レスポンスヘッダーで確認できます。モデルやプロバイダーの候補をヘッダーで制限することもできるので、使ってよい範囲を決めたうえで任せられます。

https://developers.cloudflare.com/ai-gateway/features/auto-router/

Auto Router 自体は Beta 期間中無料です。選ばれたモデルの利用料とは別なので、注意が必要です！（使っただけかかる）

## モデルの使いすぎを調べる User Insights

AI Gateway の User Insights にも、タスクの内容やモデルの選び方を分析する機能が追加されています。

https://blog.cloudflare.com/ai-model-overuse-user-insights/

「誰が何トークン使ったか」に加えて、コーディング、調査、文章作成、要約など、何のために使っているかを確認できます。タスクに対して高性能すぎるかもしれないモデルの利用や、完了までに何ターンかかっているかも見られます。

![AI Gateway の User Insights で、タスクごとのモデルと推定のコスト削減候補を表示する画面](/images/cloudflare-birthday-week-2026-day3/user-insights-potential-savings.webp)
*Potential Savings の表示例。表示される削減額は推定値です（[Cloudflare Blog](https://blog.cloudflare.com/ai-model-overuse-user-insights/) より引用）*

たとえば短い要約にも毎回高価なモデルを使っているなら、そこを見直すきっかけになります。ただ、表示されたものがすべて無駄という判断ではなく、品質や応答時間も見ながら検討するための情報です。

AI Gateway の利用者は無料で使えます。独自アプリで今回の会話分析を使う場合は、安定した `user_id` と `session_id` をリクエストに付ける必要があります。処理は非同期で、集計に1日程度の遅れが出ることもあるため、リアルタイムの監視より、継続的な利用傾向の確認に向いています。

## Registrar のドメイン検索と API が改善

Cloudflare Registrar のドメイン検索も新しくなりました。検索した名前を、対応する420以上の拡張子で調べられるようになります。

https://blog.cloudflare.com/simplifying-domains/

以前は一部の拡張子から20件程度の候補を表示していましたが、新しい検索では取得済みのドメインも含めて表示し、絞り込みや並び替えができます。初年度の登録料と更新料も一緒に確認できます。

Birthday Week にあわせて、`.io`、`.dev`、`.app`、`.tech` など一部の拡張子では、初年度の登録料の割引も始まっています。更新料も見えるのはありがたいですね。

API には、実際の購入を発生させずに動作を確認できる Sandbox や、他社からのドメイン移管などが追加されています。MCP や `cf` からも操作でき、空き状況の確認なら以下のように書けます。

```shell:Terminal
$ cf registrar registrations check example.com
```

検索の裏側も Workers、Durable Objects、Workers KV、WebSockets で構築されています。検索セッションごとに Durable Object が結果を管理し、キャッシュや DNS、レジストリへの照会結果が揃うにつれて、該当するドメインの表示だけを更新するとのことです。

DNS にレコードがないだけでは未登録とは限らないので、速い確認と、より確かな確認を組み合わせているんですね。レジストラのドメイン検索というあんまり話を聞かない検索画面の作り方としても面白い内容でした。

## AI Agent をサイトの利用者としてどう受け入れるか

ここからは、AI Agent によるアクセスと収益化の話です。今回の発表の背景を説明する記事も公開されています。

https://blog.cloudflare.com/agentic-web/

従来は検索エンジンにクロールしてもらうことで人がサイトに来て、広告や購読につながっていました。AI が取得した記事をもとに回答を作る場合、その先で人がサイトに来るとは限りません。

一方で、情報を探したり API を使ったりする Agent は、人の代わりにサービスを使う利用者でもあります。Cloudflare は、相手を識別し、アクセス条件を決め、利用に応じた支払いまで扱う方向に進めています。

記事中には既存の AI Crawl Control や Web Bot Auth なども出てきますが、この日に Beta になった収益化の仕組みが、次の2つです。

## HTTP 402 で Agent に課金する Monetization Gateway

Monetization Gateway は、Web サイトや API、MCP ツール、データなどへのアクセスに料金を設定する仕組みです。**現時点では米国の対象販売者・購入者向けの Closed Beta** で、Dashboard から参加を申請します。

https://blog.cloudflare.com/monetization-gateway-beta/

支払いが必要なリクエストに `HTTP 402 Payment Required` を返し、Agent が支払いの承認に署名して再リクエストします。Gateway が検証・決済を扱い、リソースを返す流れで、別の決済画面へ移動する必要がありません。

![Agent が HTTP 402 を受け取り、支払い承認付きで再リクエストしてレスポンスを受け取る流れ](/images/cloudflare-birthday-week-2026-day3/monetization-gateway-flow.webp =600x)
*Monetization Gateway を通じた支払いとリクエストの流れ（[Cloudflare Blog](https://blog.cloudflare.com/monetization-gateway-beta/) より引用）*

URL やヘッダー、クエリなどで課金対象を決められ、固定料金に加えて、上限額を提示して実際の利用量で精算する料金体系にも対応します。現在の決済は x402 を使い、Coinbase の Facilitator を通じて、Base 上の USDC で行われます。

すでに検索 API や PDF 生成 API などで使われているほか、Cloudflare 自身の AI Gateway でも、米国の顧客が一部モデルの推論料金をリクエスト時に支払えるようになっています。

「この API を1回だけ使いたい」という Agent に対して、アカウント作成や月額契約を挟まず販売するための仕組み、と考えるとわかりやすそうです。

https://developers.cloudflare.com/monetization-gateway/

## コンテンツが使われた分を支払う Pay Per Use

Pay Per Use も Beta になりました。こちらは AI 企業がコンテンツを取得した後、回答やサービスの中で実際に利用した分について、提供者へ支払う仕組みです。

https://blog.cloudflare.com/pay-per-use/

まず AI 企業側が「何を利用と数えるか」と単価を提示し、コンテンツを持つ側が、その条件を受け入れるか選びます。たとえば、記事の一部を回答に引用したときに支払う、といった形です。

利用が発生したら、AI 企業が日時・元の URL・イベント ID を API で報告します。Cloudflare が報告を集計して請求し、コンテンツ提供者には月次で支払います。Dashboard では購入者やドメインごとの利用件数、収益を確認できます。

ここでの利用回数は、AI 企業側からの自己申告です。Cloudflare が AI の回答をすべて監視して、自動的に利用を見つける仕組みではありません。何に使ってよいか、学習用途を認めるかなども、参加するプログラムの条件によります。

Monetization Gateway がリクエスト時の支払いを扱うのに対して、Pay Per Use は、取得したコンテンツのその後の利用を扱います。1回取得した記事が何度も回答に使われるようなケースでは、この違いが大きそうです。

## おわりに

Day3 は、Agent が動く環境から、エラーの調査、モデルの選択など、AIを主軸にしつつ開発者体験に大きく効いてくる内容が多かったように思います！

個人的には Workers の Issues と Containers の起動短縮がとても気になります。
Issues に関してはエラーが起きた前後の情報をまとめて Agent に渡せるのが、実際に使ってみると便利そうですし、特別な SDK や計装を追加しなくてよいのも嬉しいところです。

Containers は既存のクラスの保守期限も出ていますが、これまで起動時間などが制約になって行えなかったようなユースケースもいろいろ試していきたいなとおもいます！

:::message
この記事は本人が書いた文章をもとにAIの支援を得て背景調査・校正・修正等を行っています。
:::
