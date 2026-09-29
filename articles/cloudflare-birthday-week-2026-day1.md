---
title: "Cloudflare Birthday Week 2026 Day1"
emoji: "🎂"
type: "idea"
topics: ["cloudflare", "cloudflareworkers", "typescript", "vite", "rust"]
published: true
publication_name: "remotive"
---

Cloudflare Birthday Week 2026 が始まりました！Cloudflare の16周年にあわせたさまざまな発表が行われる1週間です。
この記事では Day1 の発表内容をまとめていきます！

公式な情報が見たい方は以下で公式ブログを一覧できます

https://www.cloudflare.com/birthday-week/

また、前日には Founders' Letter が今年も出てますのでぜひ

https://blog.cloudflare.com/cloudflares-2026-annual-founders-letter/

1日目から Wrangler の後継となる CLI や Vinext 1.0、EmDash 1.0 など、普段 Workers や Vite を使っていると気になる発表がいろいろとあったので、内容をまとめておきます。

## Cloudflare 全体を操作するコマンド `cf`

Cloudflare API 全体を扱う新しい CLI として `cf` が Open Beta で公開されました。Wrangler が約280の API 操作に対応していたのに対して、`cf` は3,000以上の API 操作をカバーします。Workers のデプロイだけでなく、DNS や Access、WAF なども同じ CLI から扱えるようになります。

https://blog.cloudflare.com/cloudflare-cf-cli-launch/

AI Agent が使うことをかなり意識した設計で、出力は JSON が基本です。`cf cli search` で、やりたいことを自然言語で渡してコマンドを探すこともできます。

### 設定ファイルが `cloudflare.config.ts` に

これまで `wrangler.jsonc` や `wrangler.toml` だった設定ファイルが `cloudflare.config.ts` に変わり、TypeScriptを用いて記述することができるようになります。

環境ごとに共通部分を使い回したり、型補完を見ながら Binding を追加したりできます。開発・ビルドの標準も Vite になるようです。

https://github.com/cloudflare/cf

これまで `env` などでめちゃめちゃ頑張って書き分けていた環境ごとの設定がわかりやすく書けるようになるのはめちゃめちゃ嬉しいなと思います。

ただ、まだ Workers 以外の DNS や Zone 設定はこの config ファイルには非対応 (?) のようなので注意が必要そうです

### Wrangler はどうなるのか？

Wrangler の保守は **Open Beta 終了後から18か月間** 続く予定なので、しばらくはそのまま利用することができます。

移行のためのコマンドも用意されていて、インストールと移行用に、以下のコマンドが案内されています。

```shell:Terminal
$ npm i -g cf
$ cf migrate
```

Vite を使っている Workers は新しい設定形式にまず変換され、esbuild が必要なものや Rust や Python などの Vite を利用しない Workers では、開発・デプロイ処理を引き続き Wrangler を利用するようです。

## SDK や CLI を生成する Forge

先ほどの `cf` を支えるコード生成基盤、Forge も OSS として公開されました。OpenAPI の定義を入力に、SDK・CLI・ドキュメントなどを生成するための仕組みで、Apache 2.0 ライセンスで公開されています。

https://blog.cloudflare.com/forge-open-source-generation-pipeline/

特徴的なのは、各チームの API リポジトリの CI で動かし、変更に対応した SDK や CLI の Preview をマージ前に確認できるところです。

また、OpenAPI から一度に全部を作るだけでなく、生成した TypeScript SDK を次の生成処理の入力にする、といったつなぎ方もできます。手書きのコマンドも含めて、CLI とドキュメントを揃えることを目指しています。

![OpenAPI から Forge CI、TypeScript SDK を経て cf CLI や Cap’n Web、ドキュメントへと生成処理がつながる図](/images/cloudflare-birthday-week-2026-day1/forge-generation-pipeline.webp =600x)
*生成した SDK を次の生成処理へ渡していく流れ（[Cloudflare Blog](https://blog.cloudflare.com/forge-open-source-generation-pipeline/) より引用）*

すでに `cf` の生成には使われていますが、Cloudflare の API ドキュメントや各言語の SDK への展開は今後進める予定のようです。


## Next.js を Vite で動かす Vinext が1.0

Next.js の API を Vite 上に再実装する Vinext が1.0になりました。App Router と Pages Router、両方が混在したアプリケーションに対応し、React Server Components や Server Actions、ISR などの互換性も改善されています。

https://blog.cloudflare.com/vinext-nextjs-on-vite/

元々はある種の joke というか、PoC 的なプロジェクトだと思っていたので個人的にはとても驚いています...

![Vinext の互換性テストの結果と、対応状況の推移を示したグラフ](/images/cloudflare-birthday-week-2026-day1/vinext-compatibility.webp)
*公式の互換性テストの状況。Cache Components を除いた集計（[Cloudflare Blog](https://blog.cloudflare.com/vinext-nextjs-on-vite/) より引用）*

既存プロジェクト向けには、互換性の確認と移行のコマンドも用意されていて、本気度が伺えますね...

```shell:Terminal
$ npx vinext check
$ npx vinext init
```

Pages Router のアプリケーションも対象に入っているので、既存の Next.js アプリのデプロイ先を見直したい場合にも使えそうですが、移行時には [公式の互換性情報](https://vinext.dev/compatibility) と自分のアプリで使っている機能をあわせて確認が必要そうです


また、もう一つ注目するべきは Cloudflare Workers での Prerendering と Cache Warming が行えるという点だと思います。

新しい Worker のバージョンに本番トラフィックを流す前に、そのバージョンへリクエストを送り、ページの描画とキャッシュの準備を進めます。準備ができてから本番に切り替えるので、公開直後のアクセスにもキャッシュを使えるという仕組みです。

大量のページをビルド時に全部生成して待つつらい時間を過ごすよりもこの方が確かに幸せなことはありそうだなと感じました。

https://blog.cloudflare.com/vinext-nextjs-on-vite/#pre-rendering-and-cache-warming


## EmDash 1.0 と Plugin Registry

Astro ベースの CMS、WordPress の精神的後継である EmDash も1.0になりました。

https://blog.cloudflare.com/emdash-cms-plugin-registry

管理画面からコンテンツを編集できるほか、API・CLI・組み込みの MCP Server 経由でも操作できるので、AI時代にも扱いやすいのが特徴でしょうか？
Cloudflare Blog 自体も8月に EmDash へ移行済みとのことです。

![EmDash の記事編集画面。本文の編集欄と、公開設定や言語を指定するサイドバーが表示されている](/images/cloudflare-birthday-week-2026-day1/emdash-post-editor.webp)
*EmDash の記事編集画面。（[Cloudflare Blog](https://blog.cloudflare.com/emdash-cms-plugin-registry/) より引用）*

今回あわせて公開された Plugin Registry は、Bluesky でも使われている AT Protocol をベースにしています。Plugin の配布情報やリリース履歴を作者側のアカウントで保持し、EmDash のカタログから探してインストールできる仕組みです。

CMS の Plugin Registry に AT Protocol を使うのは面白いですね。配布元の情報を保ったまま、別のカタログからも見つけられるようにする発想です。

### Plugin ごとに権限を分ける

Sandboxed Plugin は分離された環境で動き、コンテンツや外部サービスへのアクセスは、Plugin が宣言して管理者が許可した範囲に制限されます。

Cloudflare 上では Dynamic Workers、Node.js 上では別プロセスの `workerd` を使う構成で、Cloudflare 以外へのデプロイでもこの分離を利用できます。

Cloudflare で Sandboxed Plugin を使う場合は Workers Paid プランと D1 が必要です。現状 Hyperdrive 構成では利用できないので、このあたりは構成を決める際に確認したほうが良さそうです。

https://docs.emdashcms.com/deployment/plugin-sandbox/

画像を扱う Plugin に、ユーザー情報や未公開記事まで触らせる必要はないわけで、こういう制限を実行環境側でかけられるのはすごく良いなと思います。


## Rust Workers の Emscripten 対応

Rust Workers と `wasm-bindgen` で、`wasm32-unknown-emscripten` ターゲットを使うための experimental preview が公開されました。Emscripten と Workers の Node.js 互換 API を組み合わせ、ファイル操作やソケットなどを使う既存の Rust コードを Workers に持ち込みやすくする取り組みです。

https://blog.cloudflare.com/rust-workers-emscripten-target/

実行するのは、このターゲットで WebAssembly にコンパイルしたコードです。

Tokio の async runtime を動かすための実装も進んでいて、実験用のパッチとサンプルが公開されています。ただし、Tokio 側への取り込みは進行中で、通常の依存関係のまますべて動く段階ではありません。

その実例として、Rust 製 Minecraft Server の Pumpkin を Durable Object 上で動かしたというものが紹介されていました。

元々私はMinecraft Serverの運営とかをしていたりしたので、サーバーレス環境でマイクラサーバが動くのは考えられないというか、そんなことがあるのか...となりました。

ワールドの保存先は Durable Object の SQLite で、スレッドを使う処理は async task などへ変更しています。プレイヤーの接続には TCP ingress も使っていますが、inbound TCP 自体は今後の対応として紹介されています。

https://github.com/danlapid/rust-workers-minecraft

Durable Object で Minecraft のワールドを保存するの、かなり面白いですね...。実験段階ではありますが、今まで Workers に持ち込みにくかったライブラリを使えるようになるのは楽しみです。

## Kitesurf が WebMCP に対応、ターミナルでも動くように

Workers 上で動く Agent 向けブラウザ、Kitesurf の更新です。

今回 WebMCP に対応し、サイト側が公開した操作を Agent から直接呼び出せるようになりました。DOM 周りの改善や Web 標準への対応も進んでいるようです。

https://blog.cloudflare.com/kitesurf-update/

Browser Run では CDP・Playwright・Puppeteer などから利用でき、Workers の Binding から Quick Actions を使うこともできるようになっています。

また、ターミナル版が出たようです。

```shell:Terminal
$ brew install cloudflare/cloudflare/kitesurf
$ kitesurf https://blog.cloudflare.com
```

Kitty graphics protocol に対応しており、Ghostty や WezTerm などでも描画できます。対応していない環境向けには ANSI のテキスト表示もあります。Agent からページがどう見えているかを、手元のターミナルで確認する用途にも良さそうです。[ターミナル版の説明](https://blog.cloudflare.com/kitesurf-update/#kitesurf-runs-in-the-terminal-now)

![Kitesurf をターミナルから起動し、Web ページを表示する公式デモ](/images/cloudflare-birthday-week-2026-day1/kitesurf-terminal.gif)
*ターミナル上でページを表示する公式デモ（[Cloudflare Blog](https://blog.cloudflare.com/kitesurf-update/) より引用）*

Kitesurf は Beta 期間中、[アカウントごとの上限付き](https://developers.cloudflare.com/browser-run/limits/)で無料で利用できます。OSS 化も予定されているようです。

## Web の実測データを公開する BEACON

BEACON は、Cloudflare 上の大規模な10,000サイトから集めた Web パフォーマンスのデータセットです。匿名化した数十億件規模の実測データが Google BigQuery 上で公開され、毎日更新されます。

https://blog.cloudflare.com/how-fast-is-the-web/

LCP・CLS・INP といった Core Web Vitals を、国やブラウザ、OS などで分けて調べられます。ヒストグラムから P90 や P95 の近似値も計算できるので、たとえば LCP なら、表示に時間がかかっている側の傾向まで確認できます。

ドメイン名や URL パスは取り除かれており、個別のサイトを名指しで比較するデータセットではありません。

LCP や INP の内訳、SPA などで発生する Soft Navigation の計測も含まれています。公式記事には、画面表示の遅さをダウンロード時間・描画待ちなどに分けて見たり、初回表示とページ内遷移の差を比べたりする例があります。

## Vite+ 1.0 と VoidZero の近況

少し毛色が違うのと、直接的な関係はあまりありませんが VoidZero が Cloudflare に加わってから4か月間の更新も紹介されています。

https://blog.cloudflare.com/voidzero-update/

また、Vite、Vitest、Rolldown、Oxlint、Oxfmt などをまとめた Vite+ の1.0も発表されました。

https://voidzero.dev/posts/announcing-vite-plus-1-0

## おわりに

1日目からかなり開発者向けの発表が多くて、明日以降もとても楽しみですね

特に `cf` は Workers の設定や開発フローに関わるので、普段使っているプロジェクトでどう変わるかを見てみたいなと思っています。`cloudflare.config.ts` で環境ごとの差分を扱えるのは便利そうですし、Vite を標準にする流れもわかりやすいなと感じました。

個人的には Flagship のリリースを心待ちにしていますが... この1週間で来るんでしょうか...

:::message
この記事は本人が書いた文章をもとにAIの支援を得て構成・修正等を行っています。
:::
