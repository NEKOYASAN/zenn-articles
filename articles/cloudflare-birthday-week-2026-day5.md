---
title: "Cloudflare Birthday Week 2026 Day5"
emoji: "🎂"
type: "idea"
topics: ["cloudflare", "cloudflareworkers", "observability", "ai", "security"]
published: true
publication_name: "remotive"
---

Cloudflare Birthday Week 2026 の5日目、たぶん最終日です！
Day4 はデータ周りの発表が多かったですが、Day5 はログやトレースなど、作ったサービスを運用するときに使う機能がかなり増えています。

cloudflared の Quick Tunnels でのメール認証や、AI Gateway から Web 検索を呼べる API も来ました。今回も開発者向けの内容を中心に見ていきます。

Day4 の記事はこちらです。

https://zenn.dev/remotive/articles/cloudflare-birthday-week-2026-day4

公式の発表一覧は以下からどうぞ

https://www.cloudflare.com/birthday-week/

## ログ・トレース・アラートをまとめて扱う Cloudflare Observability

Cloudflare の可観測性を高めるための Observability Platform として Cloudflare Observability が発表されました。Workers やセキュリティ製品ごとに分かれていた調査の入り口を揃えて、ログの検索から通知、外部への書き出しまで扱いやすくする製品群です。

https://blog.cloudflare.com/one-observability-platform/

### ログを探す画面と SQL API が共通に

新しい Logs の画面では、HTTP リクエスト、Firewall Events、Workers、Containers、R2、AI Gateway などのデータセットを切り替えて調べられます。Workers Observability と Log Explorer が、同じ画面にまとまるイメージですね。

たとえば「このパスだけ遅い」というところからホスト名やデータセンターで絞り込み、Ray ID で個別のリクエストを見る、といった調査ができます。SQL やフィルタで検索するほか、自然言語でグラフを作る機能もあります。

https://developers.cloudflare.com/observability/logs/

共通の SQL API も Beta として公開されました。Day1 の `cf` CLI や Observability MCP からも使えるので、Agent にログを調べてもらうときの入り口にもなります。

Workers 用の Analytics SQL Binding も追加され、別途 API トークンを持たせずに Worker から分析データを問い合わせられます。自分のサービスの管理画面に利用状況を出したり、定期的にエラーを集計したりする用途にも使えそうです。

ただし、現時点の SQL API は1つのクエリにつき1つのデータセットを指定する形で、複数のデータセットをまたぐクエリは今後提供予定のようです。

https://developers.cloudflare.com/analytics/sql-api/

### Alerts で知りやすく、 Dashboard でわかりやすく

Notifications は Alerts という名前になり、SQL API が対応するデータをもとに、自分で条件を決める Custom Alerts が Beta で使えるようになりました。

「オリジンの 5xx が5分間にわたって増えた」「デプロイ後に Worker のエラーが増えた」といった条件を、しきい値・異常検知・SLO で設定できます。Webhook への通知も全プランで利用できるので、普段使っているツールや Agent に調査をつなげやすくなります。

Domain Analytics では通信量、パフォーマンス、セキュリティ、キャッシュ、オリジン、DNS の情報がまとまり、全プランで30日分の分析データを確認できます。Custom Dashboards で必要な指標を並べておくこともできます。

割とこれまで Cloudflare の Dashboard が、画面上でさまざまな場所に情報が散在してしまっていたので、これが一箇所にまとまるのはいいですね

### ログとトレースの新料金

料金も12月1日からイベントの件数ではなく、取り込んだデータ量と保存量をもとにする方式へ変わります。

Workers・Containers・AI Gateway のログ、R2 Data Access Logs、Issues、Cloudflare / Workers Traces などは、アカウント単位で以下の枠を共有します。

| 項目 | Free | Paid |
| --- | --- | --- |
| 含まれる取り込み量 | 1日あたり 0.5 GB | 請求期間あたり 50 GB |
| 含まれる保存量 | 7日間の保存込み | 請求期間あたり 12 GB-month |
| 取り込みの超過料金 | 超過分の取り込み不可 | 1 GB あたり $0.25 |
| 保存の超過料金 | 個別の従量課金なし | 1 GB-month あたり $0.10 |

※発表ブログでは Paid の保存枠が 10 GB-month とされていますが、10月2日時点の[公式料金ドキュメント](https://developers.cloudflare.com/observability/pricing/)では 12 GB-month となっています。上の表は料金ドキュメントに合わせています。

この共通枠のログ・トレースは、デフォルトの保存期間が7日です。最大1年までの保存期間の拡張が今後予定されているようです。また、サンプリングしないセキュリティ系のデータセットは共通枠の対象外で、取り込み1 GB あたり $1、30日間の保存込みという別料金です。

Day3 の Issues もこの料金体系に入るので、すでに使い始めた方は12月の変更を見ておくとよさそうです。

### Logpush が Free・Pro・Business にも

ログを R2 や外部のストレージなどへ送る Logpush が、Free・Pro・Business でもセルフサービスで利用できるようになりました。SQL でログを絞り込んだり、機密情報を消したり、形式を変えてから送る Transformers も正式提供です。

| 処理 | アカウントごとの月間利用枠 | 超過料金 |
| --- | --- | --- |
| R2・Pipelines への書き出し | 25 GB | 1 GB あたり $0.03 |
| 外部への書き出し | 25 GB | 1 GB あたり $0.10 |
| Transformers による加工 | 1 GB | 1 GB あたり $0.04 |

加工してから送る場合は、加工と書き出しの両方が計算されます。保存先の R2 などの料金も別です。

なお、Workers Trace Events を送る Workers Logpush は、従来の Workers Paid 向け・リクエスト数ベースの料金が続きます。OpenTelemetry 宛ての出力も、この表の GB 単位の料金には含まれません。

https://developers.cloudflare.com/logs/logpush/pricing/

Day4 の Basin と組み合わせて、Cloudflare のログを自分で集計するところまで組みやすくなりそうですね。

## Cloudflare を通るリクエストを追える Traces が Open Beta に

Observability の更新の中でも、Cloudflare Traces は気になる機能です。Worker の中だけでなく、その前後にあるルールの評価、URL の書き換え、キャッシュ、ルーティング、オリジンへの接続など、対応する処理を1本のタイムラインで追えるようになります。

https://blog.cloudflare.com/cloudflare-tracing/

「WAF に止められたのか」「Worker に届く前に URL が変わったのか」「キャッシュが効かずオリジンの応答を待っていたのか」といったことを、個別のリクエストから確認できます。

![Cloudflare Traces でキャッシュとオリジンへの処理時間を確認している画面](/images/cloudflare-birthday-week-2026-day5/traces-cache.webp)
*全体の約539msのうち、オリジンからの応答を得るまでに約528msかかっている例（[Cloudflare Blog](https://blog.cloudflare.com/cloudflare-tracing/) より引用）*

ドメインごとに有効化し、たとえば通常は1%だけ記録しておいて、調べたいパスや特定のヘッダーが付いた通信だけ100%記録する、といった設定を Trace Rules で作れます。すべての通信を常に記録しなくても、調べたいところを細かく見られるのはよいですね。

W3C 標準の `traceparent` を受け取ったり、オリジンへ引き継いだりする設定もあります。Cloudflare とアプリケーションの Span を同じ OpenTelemetry 対応の基盤へ送れば、両方をつなげて調査できます。オリジン内部まで追うには、アプリケーション側にも計装を入れる必要があります。

ちなみに、設定の挙動をシミュレーションする Rules にある既存の [Cloudflare Trace](https://developers.cloudflare.com/rules/trace-request/) と、実際の通信を記録する Observability 今回の Cloudflare Traces は別の機能です。名前がかなり似ていてわかりにくいのでなんとかして...

DDoS ルールや Access などへの計測対象の拡張は今後追加されるため、今回の段階ですべての処理が見えるわけではありませんが、Cloudflare のどこで時間がかかったのかを追いやすくなるのは嬉しいです。

https://developers.cloudflare.com/observability/traces/

## Quick Tunnels にメール認証を付けられるように

ngrok のようにローカルで動かしているアプリを、一時的な `trycloudflare.com` の URL で共有できる Quick Tunnels に、メール認証が追加されました。`cloudflared` の 2026.9.3 以降で利用でき、無料です。

https://blog.cloudflare.com/protected-quick-tunnels/

使い方は、いつものコマンドに `--allowed-mail` を足すだけです。

```bash
cloudflared tunnel --url http://localhost:5173 \
  --allowed-mail me@example.com
```

アクセスした人はメールアドレスを入力し、届いたワンタイム PIN で認証します。公開する側も見る側も、Cloudflare アカウントは不要です。

複数人を許可する場合はフラグを繰り返し指定でき、`--allowed-mail '*@example.com'` でドメイン単位の許可もできます。

![Cloudflare Access がメールを確認し、ローカルの cloudflared が許可リストと照合してアプリへ通す流れ](/images/cloudflare-birthday-week-2026-day5/quick-tunnels-auth.webp)
*メールの本人確認は Cloudflare Access、許可リストとの照合は手元の cloudflared が担当します（[Cloudflare Blog](https://blog.cloudflare.com/protected-quick-tunnels/) より引用）*

許可するメールアドレスの一覧は手元の `cloudflared` が持つので、Cloudflare 側にそのリストを登録する必要もありません。プロセスを止めればアクセスも終了します。

このフラグを付けなければ、従来どおり URL を知っている人がアクセスできる公開の Tunnel になります。

メール認証はブラウザでの操作が前提で、非対話の API クライアント向けではありません。また、Quick Tunnels 自体は開発・テスト用で、固定ホスト名や SSE には対応していません。ローカルの画面をスマホで確認したい、誰かに少しだけ見てもらいたい、という場面で便利そうです。

https://developers.cloudflare.com/tunnel/get-started/quick-tunnels/

## AI Gateway から Web Search API を呼べるように

AI Gateway 経由で Web 検索を使う Web Search API が Open Beta になりました。最初の対応プロバイダは Ceramic.ai、Exa、Linkup の3つです。

https://blog.cloudflare.com/introducing-web-search-api/

検索するとタイトル、URL、説明文が共通の形式で返ってくるので、その内容をモデルに渡して、最新の情報を参照しながら回答させられます。Day4 の AI Search が自分で取り込んだデータを検索するものだったのに対し、こちらは Web 上の情報を探す API ですね。

REST API のほか、Workers の AI Binding からも使えます。`AI` という名前の Binding と、AI Gateway のクレジットまたはプロバイダの API キーを用意しておけば、呼び出し部分はこのぐらいです。

```ts
const response = await env.AI.websearch({
  gatewayId: "default",
  query: "Cloudflare Workers の最新アップデート",
  provider: "exa",
  limit: 5,
});

const results = await response.json();
```

https://developers.cloudflare.com/web-search/how-to-use/

検索の料金は AI Gateway のクレジットから支払え、Cloudflare による上乗せはありません。既存の API キーを持ち込む BYOK にも対応しています。

| プロバイダ | Web Search API の1,000リクエストあたりの料金 |
| --- | --- |
| Ceramic.ai | $0.25 |
| Exa | $7.00 |
| Linkup | $5.00 |

詳細は[公式プロバイダ一覧](https://developers.cloudflare.com/web-search/providers/)を見ていただければと思いますが、Exa は `auto`、Linkup は `fast` での検索になるなど、使う検索モードにも違いがあります。BYOK の場合は各プロバイダとの契約に従って直接請求されます。

検索もモデルへのリクエストと同じ AI Gateway のログで追えるのは、Agent が何を調べて回答したのかを確認するときによさそうです。

なお、AI Gateway が検索ツールの実行まで引き受ける Server Tools は今後の予定です。今はアプリケーション側で検索を呼び出し、結果をモデルに渡す処理を組む形になります。

## Enterprise 向けだった機能を、もっと広く使えるように

昨年から進めている「Cloudflare の機能を全ユーザーが使えるようにする」という取り組みの進捗も公開されました。今回の Logpush の開放に加えて、複数アカウントの管理や、この1年の制限緩和が紹介されています。

https://blog.cloudflare.com/enterprise-for-all-update/

Dashboard の New Account ボタンから追加のアカウントを作れるようになり、プロジェクトやチームごとにリソースと請求を分けられます。Workers 単位でのロールベースアクセスコントロールも加わっているので、アカウントを分けるほどではない場合にも、担当する Worker だけ権限を渡しやすくなります。

複数のアカウントをまとめて管理する Organizations は、現時点では Enterprise 向けの Beta です。10月中の正式提供、Free アカウントへの展開は2027年初めが予定されています。（早く欲しい）

利用額を制限する Hard Spending Caps も開発中で、2026年の第4四半期に初期提供を予定しているとのことでした。予算の通知だけでなく、上限で止める選択肢が来るのは気になりますね。

## OHTTP Gateway が Closed Beta に

Cloudflare OHTTP Gateway の Closed Beta が発表され、Waitlist の受付が始まりました。OHTTP は Oblivious HTTP の略で、アプリケーションのサーバーに利用者の IP アドレスを見せずに HTTP リクエストを届けるための仕組みです。

https://blog.cloudflare.com/announcing-cloudflare-ohttp-gateway/

間に役割の違う2つのサーバーを置きます。Relay は利用者の IP アドレスを見ますが、暗号化されたリクエストの中身は読めません。Gateway はリクエストを復号してアプリへ渡しますが、通信元として見えるのは Relay の IP アドレスです。

![第三者の OHTTP Relay から Cloudflare Access と OHTTP Gateway を経由してアプリケーションへリクエストが届く構成](/images/cloudflare-birthday-week-2026-day5/ohttp-gateway-flow.webp)
*Relay と Gateway を別の運営者に分け、利用者の IP とリクエストの内容が1か所に揃わないようにします（[Cloudflare Blog](https://blog.cloudflare.com/announcing-cloudflare-ohttp-gateway/) より引用）*

今回 Cloudflare が提供するのは Gateway 側で、Relay は別の事業者のものを組み合わせます。Cloudflare の後ろや Workers 上にアプリを置きつつ、Gateway の暗号処理や鍵の管理を任せられるようになります。以前からある Privacy Gateway は、役割をわかりやすくするため Cloudflare OHTTP Relay に改名されます。

これはネットワーク上の識別情報を分ける仕組みなので、リクエスト本文にメールアドレスなどを入れれば、アプリ側ではその人を識別できます。アプリでどの情報を送るかも含めて設計する必要があります。

## Account Abuse Protection に不正利用の調査画面が追加

Account Abuse Protection に、ログインや新規登録の動きをアカウント単位で調べる Dashboard が追加されました。まずは Early Access の利用者向けで、Bot Management Enterprise の契約者が参加を申し込めます。

https://blog.cloudflare.com/account-abuse-protection-dashboard/

メールアドレスやユーザー名などからドメインごとの Hashed User ID を作り、ログイン失敗、漏洩した認証情報との一致、IP アドレス、端末、国などの情報をまとめます。リクエスト単体を見て終わりにせず、「このアカウントは普段と違う動きをしていないか」を追えるようにするものです。

全体のログイン失敗が増えたところから、関係するアカウントを絞り込み、個別の履歴を調べる、という流れで使えます。不正が確認できた場合は、その Hashed User ID を WAF のルールに使って Challenge や Block につなげられます。

調査画面を見る権限と、メールアドレスなどの追加の個人情報を見る権限も分かれています。異常なログインがあったことと、実際にアカウントが乗っ取られたことは別なので、履歴を見ながら判断するための更新ですね。

## ネットワーク性能の測定結果も更新

ネットワーク性能の定期報告では、世界の利用者数上位1,000ネットワークのうち、Cloudflare が最速だった割合が4月の60%から8月の74%になったと発表されています。

https://blog.cloudflare.com/network-performance-birthday-week-2026/

ここで比べているのは、ブラウザからの TCP 接続にかかる時間です。接続時間の P25・P50・P75 を使う Trimean という指標で、CloudFront、Google、Fastly、Akamai などと比較しています。ページ全体の表示速度や、すべての地域・回線で必ず一番速い、という数字ではありません。

今回は従来のエラーページに加え、一部の Free の Challenge Pages でもバックグラウンドで測定するようになりました。測定できる利用者や回線の幅が増えたことも順位の変化に関係しているため、60%から74%への変化を、そのまま通信速度の改善率として読むものではないですね。

## おわりに

これで Birthday Week 2026 の5日間が終わりました！

最終日は Observability の更新が多く、Day3 の Issues と合わせて、作ったアプリのエラーを見つけて原因を調べるところがかなり揃ってきた印象です。Cloudflare 上のルール処理なのか、Worker なのか、オリジンなのかを Traces で追えるのは特に気になります。

この5日分だけでも試したいものがだいぶ増えてしまいましたが、まずは身近な開発環境から触ってみたいと思います！

でも Flagship のリリースはずっと待ってます....

:::message
この記事は AI の支援を得て情報収集・執筆・校正を行っています。
:::
