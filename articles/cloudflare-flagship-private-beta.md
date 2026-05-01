---
title: "Cloudflare の Feature Flag サービス「Flagship」を試す"
emoji: "🚩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Cloudflare", "Web"]
published: true
---

Cloudflare は Agent Week の最終日に Flagship という Feature Flag SaaS の代替のようなものを発表しました。

先日、ありがたいことに Private Beta に招待いただいたので、触ってみた感触と使い方みたいなところをまとめます。


:::message
**おことわり**

Private Betaで提供されている時点でのものです。正式リリース時・Public Beta時には異なる場合があります。
:::

## Flagship とは？

OpenFeatureに準拠したAPIを提供する、Cloudflare ネイティブな Feature Flag サービスです。

Cloudflare Workers では Binding を用いて、ブラウザなどではAPIを経由して利用できるため、多くのユースケースで利用できます。

特に Workers 上ではフラグ評価が極めて低レイテンシで利用できるため、Workers 向けに最適化された Feature Flag だと考えることができます。

詳細は Agent Week に公開された Cloudflare 公式のブログをご覧ください。

https://blog.cloudflare.com/ja-jp/flagship/

## 使ってみる

### Cloudflare Dashboard から始める

Dashboard の Compute の下に「Flagship」が追加されます

![Flagshipページに初めてアクセスした様子。なにもアプリケーションが作成されていないため空](/images/cloudflare-flagship-private-beta/flagship-empty.png)

「Create app」 から app group を作成します

今回は `flagship-first-example` という名前で作成します

![Create app を押した様子。app nameを入力する欄があり、flagship-first-exampleと入力している](/images/cloudflare-flagship-private-beta/create-app-name.png)

続いて、Flagを一つ作成します。

お試しで `enable-about-page` っていうフラグにしましょう。

![continueボタンを押した後の様子。flag keyとDescriptionを入力する欄があり、enable-about-pageと入力した](/images/cloudflare-flagship-private-beta/create-first-flag.png)


続いて Variation を選択できます。（UnleashなどでいうVariantとかと似たものですかね）

今回は enable or disable を表現したいので、boolean を選択します。また、デフォルトは disable であってほしいので、false を Default にします

![Flagの作成中の様子。Variationを選択する欄があり、booleanを選択した](/images/cloudflare-flagship-private-beta/select-variation-in-create.png)

Createを押すとこのように作成され、利用が可能になります。

![作成が完了した様子](/images/cloudflare-flagship-private-beta/app-created.png)


### Dashboardで詳しく見る

#### Quick Start

Quick Startの欄では

Workers で Binding を直に使って flag を取得する example

![Workers で Binding を直に使って flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/binding-direct.png)

Workers で Binding と OpenFeature SDK を使って flag を取得する example

![Workers で Binding と OpenFeature SDK を使って flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/binding-openfeatures.png)

Workers で HTTP 経由で OpenFeature SDK を使って flag を取得する example（使うことあるのか...?）

![Workers で HTTP 経由で OpenFeature SDK を使って flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/http-workers.png)

JS のサーバーアプリケーション で HTTP 経由で OpenFeature SDK を使って flag を取得する example

![JS のサーバーアプリケーション で HTTP 経由で OpenFeature SDK を使って flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/http-server-js.png)

JS のクライアントアプリケーション で HTTP 経由で OpenFeature SDK を使って flag を取得する example

![JS のクライアントアプリケーション で HTTP 経由で OpenFeature SDK を使って flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/http-client-js.png)

curl で flag を取得する example

![curl で flag を取得する example が書かれた画像](/images/cloudflare-flagship-private-beta/http-curl.png)

このように基本HTTP経由の場合は、appId, accountId, authToken が必要になります。

ただ、この authToken を発行する場所がここにはなく、標準の API Token 作成ページで作成することになり、現状の機能ではAPI Tokenの範囲を特定のapp groupに制限する機能もないため、HTTP経由でClientに露出しながら使うというのはかなり厳しいと思います（Cloudflareさん！なんとかして！）


![Cloudflare の authToken を作成する画面の画像](/images/cloudflare-flagship-private-beta/token-create.png)

#### Flags

Flags ページではフラグの一覧表示・それぞれの設定変更が行えます。

![Flagsの一覧画面の画像](/images/cloudflare-flagship-private-beta/flags-page.png)

Create flag ボタンは先ほどのアプリ作成時のflag作成フローと同じものが表示されます。

Status がそれぞれの flag が利用されるかどうかを示しているわけですが、 boolean の場合は実際に返ってるステータスと混乱して若干わかりにくいなという感じはしますね。 Status 別に分かれてるとかがわかりやすいんでしょうか...

各 Flag key をクリックすると、それぞれの flag の個別ページに遷移します。

![Flag単体の画面の画像](/images/cloudflare-flagship-private-beta/flag-detail-page.png)

Variants の型に関しては後から変えることはできません。 

Boolean で Value が true/false 以外に変えられるように見えるのはバグだと思います。（実際に変更しようとするとエラーとなります。）


![変更しようとするとエラーになっている画像](/images/cloudflare-flagship-private-beta/flag-boolean-intermediate-error.png)

Label や Default に関しては自由に変更することが可能です。

Flag の名前についても flag key として利用されているため変更することはできませんが、説明文変更や削除は行えます。

Changelog はかなり魅力的な機能で、それぞれのフラグの変更履歴を確認できるので、どのタイミングでフラグを誰が変更したかをしっかりと追跡できます。

![FlagのChangelogの画像](/images/cloudflare-flagship-private-beta/flag-detail-page-changelog.png)

#### Targeting Rule

同じ画面内にはありますが、特徴的で面白い機能として Targeting Rule があります。

OpenFeature では、Flag の値を取得する際に Evaluation Context というものを指定してリクエストが行われます。
これは一定自由に設定することができ、UserId や SessionId など、何かしらの識別子を Feature Flag Provider に送信することができます。

これを用いて Targeting Rule では Flag の出し分けを実施できます。（カナリアリリースのようなものですね）

![FlagのTargeting Ruleの画像](/images/cloudflare-flagship-private-beta/flag-detail-page-targetingrule.png)

上のようにかなり柔軟性のある Rule 設定ができることがわかります。（この例だと userId が nekoyasan か Nekoya3_ ならば `true` が返り、それ以外なら default である `false` が返るという挙動になります。）

また、 Rule を設定しなくとも、一定の割合でランダムに利用可能なユーザーを作成することもできます。

![FlagのTargeting Ruleの Percentage 設定の画像](/images/cloudflare-flagship-private-beta/flag-detail-page-percentage.png)

Targeting keyをもとにセグメントを行うこともでき、自由な割合・パーセンテージを入力することができます（Totalは入力できるように見えてできない & 100％以外はエラーとなります。）

Save したあとであれば、上部の Test flag から挙動を確かめることができます。

たとえば先ほどと同じルールでやると、 `Nekoya3_` と userId に指定されている場合は true が返り（reasonも `TARGETING_MATCH` となってますね）

![FlagのTest FlagでNekoya3_を指定して、Trueがかえってきている画像](/images/cloudflare-flagship-private-beta/test-flag-nekoya3_-true.png)

`Inuyasan` と userId に指定されている場合は false が返り（reasonも `DEFAULT` となってますね）

![FlagのTest FlagでInuyasanを指定して、Falseがかえってきている画像](/images/cloudflare-flagship-private-beta/test-flag-Inuyasan-false.png)

このようにある程度柔軟な Targeting Rule を指定できるのはかなりの強みだなと感じます。

### Workers Binding から使ってみる

お試しとして Hono を使った Backend で Workers Binding 経由で利用してみます。

まずは wrangler.jsonc に Binding を追加します。

ローカルでFlagをいじる方法も現状だと不明なので、remote を true にしておきます。

```diff json:wrangler.jsonc
   "name": "flagship-example",
   "compatibility_date": "2026-04-28",
   "main": "./apps/backend/src/index.ts",
+  "flagship": [
+    {
+      "app_id": "aaf1fab0-bb42-4d02-b189-2b2a151515cf",
+      "binding": "FLAGS",
+      "remote": true,
+    },
+  ],
 }
```

Bindingしているので hono の c.env から FLAGS が利用できるようになります。

今回は `/api/user/:userId/about` にアクセスして、flagが `false` なら 404が、 `true` なら 200 で title の json が返るようにしてみます。

```diff typescript:src/index.ts
import { Hono } from "hono";

const app = new Hono<{
  Bindings: Bindings;
}>();

+app.get("/api/user/:userId/about", async (c) => {
+  const userId = c.req.param("userId");
+
+  const enableAboutPage = await c.env.FLAGS.getBooleanValue("enable-about-page", false, {
+    userId: userId,
+  });
+
+  if (!enableAboutPage) {
+    return c.notFound();
+  }
+
+  return c.json({
+    title: `${userId} の About`,
+  });
+});

export default app;
```

これが今回のメインですね

```typescript:src/index.ts

  const enableAboutPage = await c.env.FLAGS.getBooleanValue("enable-about-page", false, {
    userId: userId,
  });
```

getBooleanValue のほかにもそれぞれの型ごとに関数が存在するわけですが今回はおいておいて、第一引数にflag keyを、第二引数にデフォルト値を指定します。

たとえば、flag keyを誤って削除した場合や、存在しないkeyだった場合には第二引数（今回だとfalse）が採用されます。

第三引数は OpenFeature 準拠の Evaluation Context を設定でき、今回はuserIdを渡しています。

実際にこれを起動してcurlでリクエストを送ってみると

```shell
❯ curl http://localhost:5173/api/user/Nekoya3_/about
{"title":"Nekoya3_ の About"}⏎

❯ curl http://localhost:5173/api/user/Inuyasan/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/nekoyasan/about
{"title":"nekoyasan の About"}⏎
```

このように、Targeting Ruleで指定した挙動で動作していることがわかります。

これを 50% / 50% の percentage splitにすると

```shell
❯ curl http://localhost:5173/api/user/Nekoya3_/about
{"title":"Nekoya3_ の About"}⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
{"title":"Nekoya3_ の About"}⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
{"title":"Nekoya3_ の About"}⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/Nekoya3_/about
404 Not Found⏎
```

このように一定ランダムに返ってくるようになります。

また、userId を Targeting Key に指定すると、


![FlagのTargeting KeyをuserIdにした様子](/images/cloudflare-flagship-private-beta/flag-targeting-key-userId.png)

```shell 
❯ curl http://localhost:5173/api/user/1/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/2/about
{"title":"2 の About"}⏎

❯ curl http://localhost:5173/api/user/3/about
{"title":"3 の About"}⏎

❯ curl http://localhost:5173/api/user/4/about
{"title":"4 の About"}⏎

❯ curl http://localhost:5173/api/user/5/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/6/about
{"title":"6 の About"}⏎

❯ curl http://localhost:5173/api/user/7/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/8/about
{"title":"8 の About"}⏎

❯ curl http://localhost:5173/api/user/9/about
{"title":"9 の About"}⏎

❯ curl http://localhost:5173/api/user/9/about
{"title":"9 の About"}⏎

❯ curl http://localhost:5173/api/user/9/about
{"title":"9 の About"}⏎

❯ curl http://localhost:5173/api/user/9/about
{"title":"9 の About"}⏎

❯ curl http://localhost:5173/api/user/7/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/7/about
404 Not Found⏎

❯ curl http://localhost:5173/api/user/7/about
404 Not Found⏎
```

このように同一IDであれば同じflagが返るようになるため、カナリアリリースなどに利用しやすくなります。

これだけでも十分 Feature Flag Provider として利用できることがわかったかと思います。

### フロントエンド(Browser) から使ってみる

次はフロントエンドから利用してみます。

まずは必要なライブラリ（`@openfeature/web-sdk`, `@cloudflare/flagship`）をインストールします

```shell
❯ vp add @openfeature/web-sdk @cloudflare/flagship
Progress: resolved 395, reused 0, downloaded 0, added 0, done

dependencies:
+ @cloudflare/flagship ^0.2.0
+ @openfeature/web-sdk ^1.8.0

Packages: +3
+++
. prepare$ vp config
└─ Done in 394ms
Done in 5.4s using pnpm v10.33.2
```

clientを取得するutilを用意します

```typescript:flags.ts
import { OpenFeature } from "@openfeature/web-sdk";
import { FlagshipClientProvider } from "@cloudflare/flagship/web";

export const getClient = async (context: { userId: string }) => {
  await OpenFeature.setProviderAndWait(
    new FlagshipClientProvider({
      appId: "aaf1fab0-bb42-4d02-b189-2b2a151515cf",
      accountId: "xxxxxxxxxxxxxxxxxxxxxxxxxxx",
      authToken: "your-token",
      prefetchFlags: ["enable-about-page"],
    }),
  );
  await OpenFeature.setContext(context);
  return OpenFeature.getClient();
};
```

Tanstack Routerを利用しているのでloaderでclientを作り、Component側でgetBooleanValueします。

```tsx:$userId.tsx
import { getClient } from "#/utils/flags";
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/user/$userId")({
  loader: {
    handler: async ({ params }) => {
      const flagClient = getClient({ userId: params.userId });
      return flagClient;
    },
  },
  component: RouteComponent,
});

function RouteComponent() {
  const { userId } = Route.useParams();
  const flagClient = Route.useLoaderData();

  const enableAboutPage = flagClient.getBooleanValue("enable-about-page", false);

  if (!enableAboutPage) {
    return <div>{userId} さんのトップページ</div>;
  }

  return (
    <div>
      {userId} さんのトップページ <a href={`/user/${userId}/about`}>Aboutページへ</a>
    </div>
  );
}
```

これで動作すると思いきや、CORSの影響で動作しませんでした。

何か修正が必要なのかまだClient側のFlagは未対応なのかわかりませんが、続報を待ちたいと思います。

![ブラウザでCORSエラーになっている様子](/images/cloudflare-flagship-private-beta/flag-browser-cors-error.png)

## おわりに

Cloudflare の Feature Flag サービスである Flagship の Beta を試してみました。

Bindings 経由で Workers 上の利用は凄く簡単で、かつUIも扱いやすいためとても良く素晴らしいものだと感じています。

Client (Browser) での例は上手く動作せず少し残念ではありましたが、今後に期待...という感じかなと思っています。

Publicになった際にはぜひ皆さんもご利用ください！

Client周りなどで変化があれば追記します！

## 参考
https://github.com/cloudflare/flagship/tree/main

https://blog.cloudflare.com/ja-jp/flagship/

https://openfeature.dev/

https://developers.cloudflare.com/flagship/get-started/
