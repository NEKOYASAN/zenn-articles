---
title: "Cloudflare Workers に SPA アプリケーションをデプロイする"
emoji: "🌩️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cloudflare", "cloudflareworkers", "typescript", "vite"]
published: true
publication_name: "mierune"
---

SPA アプリケーションをデプロイするのにずっと、Cloudflare Pages を使っていましたが、Cloudflare 公式のドキュメントにも 「新しいプロジェクトにはCloudflare PagesよりもCloudflare Workersの使用を推奨しています。」という記述が追加されたので、Cloudflare Workers に移行することにしました。
移行ドキュメントがかなりわかりやすいのであまり困ることはないかもしれませんが、新たにプロジェクトを作成してデプロイする術を残しておきます。
よく利用されるフレームワークはフレームワークのガイドに書かれていたりしますのでそちらをご確認ください！

https://developers.cloudflare.com/workers/static-assets/migration-guides/migrate-from-pages/

## 手順
サンプル的なプロジェクトを作成してありますので、コードをご覧になりたいかたはこちらからどうぞ！
https://github.com/NEKOYASAN/vite-spa-workers-deploy-example

### 1. SPA アプリケーションの作成
Vite + React の SPA アプリケーションを作成します。

```shell:Terminal
$ pnpm create vite@latest vite-spa-workers-deploy-example
◇  Select a framework:
│  React
│
◇  Select a variant:
│  TypeScript + SWC
│
◇  Scaffolding project in /home/Developer/nekoyasan/vite-spa-workers-deploy-example...
│
└  Done. Now run:

  cd vite-spa-workers-deploy-example
  pnpm install
  pnpm run dev

$ cd vite-spa-workers-deploy-example
$ pnpm install
$ pnpm run dev
```

こんなふうないつも通りの画面がみえればOKです

![vite-spa-init.png](/images/cloudflare-workers-static-spa-deploy/vite-spa-init.png)

ここまで
[https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/14330c30454fcdac6f6ab3f6ddbc96ef073c9c18](https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/14330c30454fcdac6f6ab3f6ddbc96ef073c9c18)

### 2. Project への Wrangler のセットアップ

#### Wrangler CLI をインストール

```shell:Terminal
$ pnpm add -D wrangler
```

#### Wrangler の設定ファイルを作成します
一気に設定ファイルつくるような便利なコマンドはないのでサクッとファイルつくります

```shell:Terminal
$ touch wrangler.jsonc
```

`wrangler.jsonc` の中身は以下のようにします

```json:wrangler.jsonc
{
  "name": "vite-spa-workers-deploy-example",
  "compatibility_date": "2025-04-01",
  "assets": {
    "directory": "./dist/"
  }
}
```

assets.directory には、Vite でビルドしたファイルの出力先を指定します。
この assets に指定したディレクトリは Static Assets として Cloudflare Workers にデプロイされ、Workers の起動・課金対象になりません。

#### `package.json` にスクリプトを追加
あわせて便利なので `package.json` にスクリプトを追加しておきましょう

```diff json:package.json
diff --git a/package.json b/package.json
index b38dc6c..cf59680 100644
--- a/package.json
+++ b/package.json
@@ -7,7 +7,8 @@
     "dev": "vite",
     "build": "tsc -b && vite build",
     "lint": "eslint .",
-    "preview": "vite preview"
+    "preview": "wrangler dev",
+    "deploy": "pnpm run build && wrangler deploy"
   },
```

お試しで `preview` を確認します

```shell:Terminal
$ pnpm run build

> vite-spa-workers-deploy-example@0.0.0 build /home/Developer/nekoyasan/vite-spa-workers-deploy-example
> tsc -b && vite build

vite v6.3.4 building for production...
✓ 32 modules transformed.
dist/index.html                   0.46 kB │ gzip:  0.30 kB
dist/assets/react-CHdo91hT.svg    4.13 kB │ gzip:  2.05 kB
dist/assets/index-D8b4DHJx.css    1.39 kB │ gzip:  0.71 kB
dist/assets/index-9_sxcfan.js   188.05 kB │ gzip: 59.21 kB
✓ built in 576ms

$ pnpm run preview

> vite-spa-workers-deploy-example@0.0.0 preview /home/Developer/nekoyasan/vite-spa-workers-deploy-example
> wrangler dev


 ⛅️ wrangler 4.13.2
-------------------

No bindings found.
⎔ Starting local server...
[wrangler:inf] Ready on http://localhost:8787

```

アクセスすると普通に見れてますし、Counterも動きますね
![preview-command-check.png](/images/cloudflare-workers-static-spa-deploy/preview-command-check.png)

ここまで
[https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/d47d411306932a4ef49f83236b61dd642d7b3467](https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/d47d411306932a4ef49f83236b61dd642d7b3467)

### 3. Workers へのデプロイ 
Wrangler に未ログインの場合は `wrangler login` でログインしておいてください

```shell:Terminal
$ wrangler deploy

> vite-spa-workers-deploy-example@0.0.0 deploy /home/Developer/nekoyasan/vite-spa-workers-deploy-example
> pnpm run build && wrangler deploy


> vite-spa-workers-deploy-example@0.0.0 build /homeDeveloper/nekoyasan/vite-spa-workers-deploy-example
> tsc -b && vite build


vite v6.3.4 building for production...
✓ 32 modules transformed.
dist/index.html                   0.46 kB │ gzip:  0.30 kB
dist/assets/react-CHdo91hT.svg    4.13 kB │ gzip:  2.05 kB
dist/assets/index-D8b4DHJx.css    1.39 kB │ gzip:  0.71 kB
dist/assets/index-9_sxcfan.js   188.05 kB │ gzip: 59.21 kB
✓ built in 576ms

 ⛅️ wrangler 4.13.2
-------------------

🌀 Building list of assets...
✨ Read 6 files from the assets directory /home/Developer/nekoyasan/vite-spa-workers-deploy-example/dist
🌀 Starting asset upload...
🌀 Found 5 new or modified static assets to upload. Proceeding with upload...
+ /index.html
+ /assets/react-CHdo91hT.svg
+ /assets/index-D8b4DHJx.css
+ /assets/index-9_sxcfan.js
+ /vite.svg
Uploaded 2 of 5 assets
Uploaded 3 of 5 assets
Uploaded 5 of 5 assets
✨ Success! Uploaded 5 files (4.43 sec)

Total Upload: 0.34 KiB / gzip: 0.24 KiB
No bindings found.
Uploaded vite-spa-workers-deploy-example (10.20 sec)
Deployed vite-spa-workers-deploy-example triggers (0.68 sec)
  https://vite-spa-workers-deploy-example.nekoyasan.workers.dev
Current Version ID: 6ed75fab-6fef-4322-91fc-e6cef5d65e0b
```

これでデプロイは完了です！なんて簡単なんでしょう...

404エラー や trailing slashes に関しては各種フレームワーク・ルーティングライブラリによっていろいろと変わってくると思いますので、以下のドキュメントをご覧いただくのが良いと思います！

https://developers.cloudflare.com/workers/static-assets/routing/

（SPAであれば、`not_found_handling = "single-page-application"`で基本的には良いはずですが...404.htmlをつくる場合は `"404-page"` です）
## おまけ

### Pages Functionsっぽいことをする

Advanced な `_worker.js` をつくっていた人向けですが、Workersになったことで自由度が広がり直感的になりました。

#### Worker の TypeScript ファイルを作成する

```ts:worker/index.ts
import { WorkerEntrypoint } from "cloudflare:workers";

export default class extends WorkerEntrypoint {
	async fetch(request: Request) {
		if(request.method !== "GET") {
			return new Response("Method Not Allowed", {
				status: 405,
				statusText: "Not Allowed",
			})
		}
		const url = new URL(request.url);
		if (url.pathname === "/api/now") {
			return new Response(new Date().toISOString());
		}
		return new Response("Not Found", {
			status: 404,
			statusText: "Not Found"
		})
	}
}
```

#### `wrangler.json` を設定する

```diff json:wrangler.json
diff --git a/wrangler.jsonc b/wrangler.jsonc
index a7ea0e1..30bd04a 100644
--- a/wrangler.jsonc
+++ b/wrangler.jsonc
@@ -1,6 +1,7 @@
 {
   "name": "vite-spa-workers-deploy-example",
   "compatibility_date": "2025-04-01",
+  "main": "./worker/index.ts",
   "assets": {
     "directory": "./dist/"
   }
```

これでStatic でヒットしないときはWorkerが使われるようになりました。

せっかくなのでReact 側も書き換えたものをDeployしてあります

ここまで
[https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/fc2aea80eed56753951091967e2214b86676f926](https://github.com/NEKOYASAN/vite-spa-workers-deploy-example/commit/fc2aea80eed56753951091967e2214b86676f926)

## おわり
今回は紹介しませんでしたがPages同様にCI/CDを設定することも出来ますし、PullRequestのPreviewも利用できます

https://developers.cloudflare.com/workers/ci-cd/builds/#get-started

また、WorkersなのでBinding経由でKVやDO、D1、R2なども利用できるので幅も大きく広がります

https://developers.cloudflare.com/workers/runtime-apis/bindings/

ぜひこれを機に Cloudflare Workers を使ってみてはいかがでしょうか！