---
title: "Cloudflare Birthday Week 2026 Day4"
emoji: "🎂"
type: "idea"
topics: ["cloudflare", "cloudflareworkers", "ai", "r2", "git"]
published: true
publication_name: "remotive"
---

Cloudflare Birthday Week 2026 の4日目です！
Day3 は Containers や Issues などWorkersのランタイムや管理に関する部分がメインでしたが。Day4 は KV Instant、Basin、K2 とデータ周りの発表が多いですね。

AI Search の正式提供や、Cloudflare 自身が学習したモデルの公開もありました。今回も開発者向けの内容を中心に見ていきます。

Day3 の記事はこちらです。

https://zenn.dev/remotive/articles/cloudflare-birthday-week-2026-day3

公式の発表一覧は以下からどうぞ

https://www.cloudflare.com/birthday-week/

## Workers KV に高速な設定配信用のモード、KV Instant が追加

Workers KV に、Quicksilver を使う新しいモードの KV Instant が発表されました。現時点では Private Beta で、利用には申し込みが必要です。

https://blog.cloudflare.com/workers-kv-instant/

Quicksilver は、Cloudflare が自社サービスの設定を世界中に配るために使っている Key-Value Store です。今回、その仕組みを Workers KV と同じ API で使えるようになります。

通常の KV は、まだキャッシュされていないデータを読むと時間がかかったり、変更がキャッシュに反映されるまで待つ必要があったりします。KV Instant は変更を各拠点に配っておくため、最初の読み取りでも速く、更新時もキャッシュの TTL が切れるのを待たずに反映されます。

公式の測定では、読み取りの P99 が 1.62ms、全拠点への書き込み反映の P99 が 256ms とのことです。反映にかかる時間がゼロになるわけではありませんが、公開フラグや機能の設定を、リクエストのたびに確認する用途にはかなりよさそうです。

`get()` や `put()` などの書き方は従来の KV と共通ですが、Namespace の作成時に `mode: "instant"` を指定します。Metadata は使えず、`list()` はページ分割せずに条件に合うキーをすべて返す、といった違いもあります。

KV Instant は、少量の設定を大量に読む用途に合わせた料金になっています。

| 項目 | 従来の KV（Paid の超過料金） | KV Instant の料金 |
| --- | --- | --- |
| 読み取り | 100万キーあたり $0.50 | 100万キーあたり $0.20 |
| 書き込み・削除・一覧取得 | 100万操作あたり $5.00 | 1操作あたり $0.10 |
| ストレージ | 1 GB あたり月額 $0.50 | 1 MB あたり月額 $100 |

従来の KV は Workers Paid に、読み取りが月1,000万キー、書き込み・削除・一覧取得がそれぞれ月100万操作、保存容量が1 GB 含まれています。表はその枠を超えた分の料金です（[公式料金表](https://developers.cloudflare.com/kv/platform/pricing/)）。

読み取りは従来の KV の従量料金より60%安い一方、書き込みや保存容量はかなり高くなります。書き込み・削除はキーごとに1操作、一覧取得は1リクエストで1操作です。

1つの Namespace に保存できるのは最大1 MB・10,000組の Key-Value で、書き込みも Namespace 全体で毎秒1回までです。ユーザーごとのデータや頻繁に変わる値をまとめて入れる用途より、全リクエストで参照する少数のフラグや設定に使うもの、と考えるとわかりやすそうですね。

## データの取り込みから SQL での分析まで扱う Basin が正式提供

Cloudflare Data Platform が、Cloudflare Basin という名前で正式提供になりました。R2 と Apache Iceberg を使い、イベントの取り込み、テーブルの管理、SQL での分析をまとめて扱うサービスです。

https://blog.cloudflare.com/cloudflare-basin/

これまで別々の名前で出ていたサービスも、Basin の名前に揃います。

| これまでの名前 | 新しい名前 | 主な役割 |
| --- | --- | --- |
| Cloudflare Pipelines | Basin Pipelines | イベントを受け取り、SQL で加工して保存する |
| R2 Data Catalog | Basin Catalog | Iceberg のテーブル情報とメンテナンスを管理する |
| R2 SQL | Basin SQL | 保存したデータを SQL で分析する |

既存のリソースや設定は、そのまま動き続けるとのことです。

たとえばアプリケーションの操作履歴を Workers から送り、不要な項目を Pipelines で取り除いて保存し、プラン別の利用状況を SQL で集計する、といった流れを組めます。Cloudflare Logpush のログも取り込めるので、エラーレスポンスだけを残して調べるような使い方もできます。（Issuesがきたのでこういうケースは減るかもしれないですが...）

Catalog は、小さなファイルをまとめる処理や古いスナップショットの削除などを自動で行います。Basin SQL も JOIN、ウィンドウ関数、JSON 関数などへの対応が進んでいて、単純なログ検索から、複数のテーブルを組み合わせた分析まで扱えるようになっています。

保存形式が Apache Iceberg なので、DuckDB や Spark など、対応する外部のツールからも同じデータを使えます。R2 のデータ転送に Egress 料金がかからないのも、ツールを選びやすくなる点ですね。取り込みや保存、クエリなどの利用料は別にかかります。

これまで BigQuery とかに保存していたのを若干 Cloudflare 内に寄せやすくなるのかな...? GIS 系のデータ入れてみてどんな感じか見てみたいですね！

https://developers.cloudflare.com/basin/

## イベントを保存して、複数の処理から読める K2

Cloudflare K2 というイベントストリーミングのサービスも登場しました。Workers Paid のアカウント向けの Public Beta で、Beta 期間中の K2 利用料は無料です。

https://blog.cloudflare.com/cloudflare-k2-streams/

イベントをストリームに順番に記録しておき、読み取る側がそれぞれのペースで処理できます。たとえば注文が確定したイベントを、売上分析と不正検知の両方に渡すような用途です。片方の処理が遅れても、保存期間内なら、残してあるイベントをあとから読み進められます。

まぁ、要するに Kafka （みたいな）ものですね。

中身は R2 上に作られたログで、もともとは Basin Pipelines の取り込み用バッファとして作られたそうです。送信側は HTTP API や Workers の Binding を使えます。

読み取りには Subscription を作ります。1つの Subscription を複数の Consumer で共有すると処理を分担でき、用途ごとに別の Subscription を作ると、それぞれが同じイベントを独立して読めます。読み終えてもイベント自体はすぐには消えず、設定した保存期間に従って保持されます。

### Queues や Pipelines とはどう使い分けるか

イベントストリーミングと聞いて Queue とか Basin Pipelines があるじゃんと思うかたもいると思います。（私もそう）

使い分けとしては、画像変換やメール送信のように、1件ずつ仕事を渡して、失敗したものを再試行するなら Queues が向いています。K2 は大量のイベントをまとめて運び、複数の処理から読んだり、過去の分を読み直したりする用途です。再配信もバッチ単位で扱います。

また、最終的に R2 のファイルや Iceberg テーブルへ保存したいなら、加工と保存まで引き受ける Basin Pipelines を使うのが公式のおすすめです。保存先や処理の内容を自分で組みたいときに K2 を選ぶ、という感じの分け方ですね。

Beta 時点ではアカウント全体の保存容量が10 GB、保存期間はデフォルト7日、通常の上限が30日です。また、初期リリースの書き込みレイテンシは P99 で約1秒とされているので、低遅延が必要な処理では注意が必要です。

既存の Kafka クライアントとの互換性や、Workers の Consumer にイベントを Push する機能は今後の予定ですので、今後に期待ですね。

https://developers.cloudflare.com/k2/

## AI Search が正式提供、画像検索と PDF の OCR も強化

AI Search が正式提供になりました。データの取り込みから検索までを管理してくれるサービスで、今回は画像やスキャンした PDF の扱いが強化されています。

https://blog.cloudflare.com/ai-search-ga/

これまでの画像検索は、画像から説明文を作り、その文章を検索する仕組みでした。今回の Qwen3-VL-Embedding では、画像そのものもベクトル化します。説明文だけでは落ちやすい色や形、模様なども検索に使えるようになります。

「似た見た目の商品を探す」「このスクリーンショットに近い画面を探す」といった使い方がしやすくなりそうですね。ECサイトにめちゃいいかも？

テキスト専用の Embedding モデルを選んだ場合は、従来どおり画像を説明文に変換して検索します。

検索時にはベクトル検索とキーワード検索を組み合わせ、必要に応じて結果を並べ替えます。見つかった部分を検索結果として返すことも、生成モデルに渡して回答を作らせることもできます。

![AI Search で検索語の処理、ベクトル検索とキーワード検索、結果の統合、回答生成を行う流れ](/images/cloudflare-birthday-week-2026-day4/ai-search-query.webp =600x)
*AI Search の検索処理の流れ。図中の各処理は設定に応じて利用されます（[Cloudflare Blog](https://blog.cloudflare.com/ai-search-ga/) より引用）*

テキストに関してもデータソースの制限に変更が入っていて、Markdown やソースコードなどのテキストファイルは、最大10 MiB まで取り込めます。PDF も OCR を有効にすると10 MiB まで扱え、スキャンしたページから文字を読み取って検索できます。OCR なしの PDF や、そのほかの変換対象形式は4 MiB のままなので、すべてのファイルが10 MiB になったわけではありません。

https://developers.cloudflare.com/ai-search/configuration/data-source/

また、正式提供に伴い AI Search の課金も11月1日に始まります。すべての Workers プランに月間の無料枠があり、取り込み500万トークン、保存10 GB-month、セマンティック検索1,000回、全文検索1,000回が含まれます。画像処理も、取り込みの500万トークンと共通の枠です。

Workers AI を使う Embedding と Reranking は AI Search の料金に含まれますが、回答生成や検索語の書き換え、外部モデルの利用料は別です。検索して回答を作るところまで全部無料になる、という意味ではないので、その点は分けて見ておきたいですね。

https://developers.cloudflare.com/ai-search/platform/limits-pricing/

## 判定に使うモデル、Clef と Clef-flash を公開

Cloudflare が学習した Decision Model (Jevみたいな) の Clef と Clef-flash が公開されました。Workers AI で利用でき、モデル自体も Apache 2.0 ライセンスで公開されています。（オープンウェイトモデル！！！）

https://blog.cloudflare.com/clef-decision-models/

Decision Model は、入力を見て「緊急か」「どの担当部署に送るか」といった判定を、確率付きの構造化データとして返すモデルです。問い合わせの内容と、担当部署の候補、判定したい質問を渡すと、それぞれの候補の確率を受け取れます。

![問い合わせの内容と質問を Decision Model に渡し、担当部署と緊急度の判定を確率付きで受け取る例](/images/cloudflare-birthday-week-2026-day4/clef-decisions.webp)
*問い合わせを担当部署へ振り分ける場合の入出力イメージ（[Cloudflare Blog](https://blog.cloudflare.com/clef-decision-models/) より引用）*

文章として回答を読ませるより、その結果をコードの分岐につなげたいときに使いやすそうです。Cloudflare 内部でも、Web サイトの内容からカテゴリを判定する用途で試しているとのことでした。

Jev の API と互換性があり、画像の入力や64kのコンテキストにも対応しています。Clef が精度を重視するモデル、Clef-flash が特に速さを重視するモデルという位置づけです。Workers の AI Binding からも `@cf/cloudflare/clef` を呼び出せます。

https://developers.cloudflare.com/workers-ai/models/clef/

### 自分の用途に合わせたモデルの調整も

Clef を自社のデータで調整する Fine-tuning のサービスも発表されました。問い合わせの振り分けなど、すでに正解付きのデータがある仕事を想定しています。

まずは Cloudflare のエンジニアチーム（FDEチーム）が個別に支援する形で始まり、その経験をもとに、データ収集から学習、再デプロイまで行えるセルフサービスの基盤を作っていくとのことです。発表タイトルには RL（Reinforcement Learning） のプラットフォームとありますが、現時点で誰でも Dashboard から学習を始められる、というものではありません。

Day3 の Containers の発表でも評価や強化学習の環境が用途に挙がっていましたが、ここにもつながってるんですかね...?たぶん...

## Artifacts が Open Beta に、次の Git プラットフォームを作るコンテストも

Git を使って操作できるストレージの Artifacts が、Workers Paid 向けの Open Beta になりました。Agent の作業ごとにリポジトリを作ったり、コードと一緒に作業の文脈を保存したりするためのサービスです。

https://blog.cloudflare.com/next-git-platform-on-cloudflare/

Workers Builds とも連携し、Artifacts のリポジトリへの Push をきっかけにビルドできます。本番ブランチへの Push はデプロイにつながり、それ以外のブランチでは Workers Previews が作られます。

![Workers の作成画面で、GitHub や GitLab と並んで Continue with Artifacts を選べるようになった画面](/images/cloudflare-birthday-week-2026-day4/artifacts-workers-builds.webp)
*Artifacts のリポジトリから Workers を作成できるように（[Cloudflare Blog](https://blog.cloudflare.com/next-git-platform-on-cloudflare/) より引用）*

Workers の Binding からリポジトリの作成・Fork、ファイルやコミットの確認、リポジトリ単位の Git トークンの発行もできます。Agent にタスクを渡すときに作業用のリポジトリを用意し、必要な範囲のトークンを渡す、といった流れをコードで組めます。

リポジトリの作成や Push などのイベントも購読できるので、変更が入ったらレビューを始めるような処理も作れます。また、Namespace の作成時に US または EU を指定して、リポジトリの保存・処理を行う場所を制限できるようになっています。（JPもください）

こちらも Open Beta になったため、リポジトリの操作と保存容量をもとに10月15日から課金が始まります。

### Artifacts を使ったプラットフォームコンペ

Workers と Artifacts を使って、Agent が並行して作業する時代の Git プラットフォームを作るコンテストも開催されています。複数の Agent が同時に変更を進められることが最低条件で、レビューやマージ、作業の文脈の残し方などをどう作るかがテーマです。

応募には5〜10分のデモ動画、MIT・Apache・BSD などのライセンスで公開したソースコード、実行手順が必要です。締め切りは10月14日で、上位3チームは各2名まで Cloudflare Connect に招待され、優勝チームには $25,000 分の Cloudflare クレジットも提供されます。

GitHub の使い方に Agent を足すのではなく、作業の進め方から考え直してほしい、という募集なのが面白いですね。リポジトリを気軽に作って使い分けられると、試せることも増えそうです。

## Cloudflare OS のマネージド版が Waitlist を開始

組織内で使う Agent の作業環境、Cloudflare OS にも更新がありました。今回は、Cloudflare がデプロイや更新などの運用を引き受けるマネージド版の Waitlist が公開されています。

https://blog.cloudflare.com/managed-cloudflare-os/

独自ドメイン、Cloudflare Access のアクセス制御、接続する AI Gateway を指定して、組織用の環境を用意できるようにするとのことです。オープンソース版はすでに公開されているので、自分の Cloudflare アカウントへのデプロイはできましたが、さらに簡単に・手軽に使えるようになりそうです。。

機能面では、既存の GitHub リポジトリを接続して、Agent にコードの調査や修正、コミット、Push、PR 作成を任せられるようになりました。Google Workspace との連携も強化され、Gmail の調査や下書き・送信、Drive 全体や特定のフォルダ・ファイルへの接続に対応しています。

作ったものを Excel、CSV、PDF、Markdown、HTML などで書き出す機能も追加されています。対応する形式は成果物によって異なり、Word の `.docx` と PowerPoint の `.pptx` への書き出しも対応予定のようです。

会社への導入となると、デプロイして終わり！とはいかずに運用も必要になるので、そこまで任せられる選択肢ができるのはありがたいですね。

## Workers の Web Crypto で耐量子暗号を試せるように

Day2 にも耐量子暗号の話がありましたが、今回は Workers のコードから使う API の更新です。Web Crypto に、鍵共有のための ML-KEM と、電子署名のための ML-DSA が追加されました。

https://blog.cloudflare.com/workers-ml-kem-ml-dsa-support/

`wrangler.jsonc` で、以下の Compatibility Flag を指定して有効にします。

```json:wrangler.jsonc
{
  "compatibility_flags": ["webcrypto_modern_algorithms"]
}
```

ML-KEM は、通信する双方が同じ秘密の鍵素材を得るための仕組みです。それだけでメッセージの暗号化まで行うものではなく、暗号化の方式と組み合わせて使います。ML-DSA は、データに署名し、その署名を検証するために使えます。

これらをランタイム側が提供するので、試すために JavaScript や Wasm の暗号実装を別途組み込む必要が減ります。秘密鍵から公開鍵を得る `getPublicKey()` や、アルゴリズムへの対応を調べる `SubtleCrypto.supports()` も追加されています。

https://developers.cloudflare.com/workers/runtime-apis/web-crypto/

## Workers AI に EuroLLM と Apertus を追加へ

AI の選択肢を増やす取り組みとして、EuroLLM と Apertus を Workers AI に提供する発表もありました。現時点ではアクセスの申請を受け付けています。

https://blog.cloudflare.com/sovereign-ai-choice-one-year-later/

EuroLLM は EU の公用語24言語を含む35言語に対応するモデルで、Apertus はスイスの研究機関が開発した多言語モデルです。利用する地域や言語に合わせて、モデルを選べるようにするという話ですね。

記事では、特定のモデルへのアクセスを失ったときに業務や防御まで止まってしまわないよう、モデルを選び直せることを重視しています。政府のサイバーセキュリティ機関や重要インフラの運営者向けに、モデルに依存しない AI 防御のワークショップも始めるとのことでした。

## おわりに

Day4 は、データを配る、集める、検索する、分析するといったところに発表が集まっていました。AI 向けの機能も多いですが、普通にアプリケーションを作るうえでも使いどころがありそうです。

個人的には KV Instant が気になります。毎回読む設定やメタデータなんかを置いておくとかが良さそうですね？

:::message
この記事は筆者本人が執筆し、AI の支援を得て情報収集・校正を行っています。
:::
