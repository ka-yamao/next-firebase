# Next.js Firebase hosting (function) へデプロイ SSR SSG のサンプル実装

2022 年 10 月 25 日に Firebase Summit 2022 で Next.js と Angular Universal を用いたサーバサイドレンダリングによる動的な Web サイトにも対応することを[発表しました。](https://firebase.blog/posts/2022/10/whats-new-at-Firebase-Summit-2022)
これまでは、SSR を使うとき Functions の設定がいろいろ面倒でした。今回から Firebase CLI ツールが自動的に Next.js や Angular Universal を認識し、Google Cloud Functions および Firebase Hosting で稼働する設定を全て自動的に行ってくれるようになりました。
早速試してみました。

- [Google の「Firebase Hosting」が Next.js と Angular Universal による動的 Web サイトもサポート、コマンド一発でデプロイ。Firebase Summit 2022](https://www.publickey1.jp/blog/22/googlefirebase_hostingnextjsangular_universalwebfirebase_summit_2022.html)

## 目次

- [環境](#環境)
- [1.Firebase CLI ツールのアップデート](#1.FirebaseCLIツールのアップデート)
- [2.フレームワークのプレビューを有効にします](#2.フレームワークのプレビューを有効にします)
- [3.Next.js のプロジェクト作成](#3.Next.jsのプロジェクト作成)
- [4.Firebase Hosting 初期化](#4.FirebaseHosting初期化)
- [5.開発モードで動かす](#5.開発モードで動かす)
- [6.SSR を実装](#6.SSRを実装)
- [7.SSG を実装](#7.SSGを実装)
- [8.Next.js ビルド](#8.Next.jsビルド)
- [9.Firebase へデプロイする](#9.Firebaseへデプロイする)
- [10.表示確認](#10.表示確認)
- [11.Firebase を確認](#11.Firebaseを確認)
- [まとめ](#まとめ)
- [デプロイのときハマった](#デプロイのときハマった)
- [参考資料](#参考資料)
- [サンプルソース](#サンプルソース)

<a id="環境"></a>
<br>
<br>

## 環境

- Node.js ： 16.14.0
- firebase（CLI ツール） ： 11.16.1
- firebase プロジェクト： Blaze 従量制 になってるプロジェクトを準備しておく
- Next：13.0.4
- React ： 18.2.0
- Yarn ： 1.17.0 ※手順は npm ではなく yarn を使います。

* ※ Node.js の環境構築、[Firebase CLI ツール](https://firebase.google.com/docs/cli) インストールの手順、Firebase プロジェクトの作成については記載しません。

<a id="1.FirebaseCLIツールのアップデート"></a>
<br>
<br>

## 1.Firebase CLI ツールのアップデート

- グローバルにインストールしていた `firebase-tools` をアップデートします。[ドキュメント](https://firebase.google.com/docs/hosting/nextjs#before_you_begin) には、11.14.2 以降と記載ありますが、不具合があるみたいで 11.16.1 のバージョンだとうまくいきました。

```
# アップデート
$ npm i -g firebase-tools to update

$ firebase --version
  11.16.1

```

<a id="2.フレームワークのプレビューを有効にします"></a>
<br>
<br>

## 2.フレームワークのプレビューを有効にします

```
$ firebase experiments:enable webframeworks
```

<a id="3.Next.jsのプロジェクト作成"></a>
<br>
<br>

## 3.Next.js のプロジェクト作成

- next-firebase というプロジェクト名で Next.js のプロジェクトを作成する。TypeScript を利用するとので `--ts` を付けてます。

```
# プロジェクトを作成するところへ移動し、Next.js のプロジェクトを作成する。next-firebase ディレクトリもできる。
$ npx create-next-app@latest --ts next-firebase

# Yesが選択されてるので Enter を押下
? Would you like to use ESLint with this project? › No / Yes

# プロジェクト直下へ移動
$ cd next-firebase

```

<a id="4.FirebaseHosting初期化"></a>
<br>
<br>

## 4.Firebase Hosting 初期化

- 予め Firebase のプロジェクトが作成済み状態で実行してます。プロジェクトは、Blaze 従量制 になっている必要があります。

```
# Hosting を初期化
$ firebase init hosting

? Please select an option: (Use arrow keys)
Use an existing project ##### すでにプロジェクトを作成しているのでここが選択された状態で Enter を押下 #####
Create a new project
Add Firebase to an existing Google Cloud Platform project
Don't set up a default project

? Select a default Firebase project for this directory: (Use arrow keys)
❯ hoge-s2s4s (hoge) ##### Blaze 従量制 になってるプロジェクトを選択 #####
  hoge-d52b8 (hoge-d52b8)

? Detected an existing Next.js codebase in the current directory, should we use this? (Y/n)
##### カレントディレクトリの Next.js があるので Y を入力して押下 #####

? Set up automatic builds and deploys with GitHub? (y/N)
##### Github で自動ビルドは、いったんなにもしないので N を入力して Enter を押下 #####

```

<a id="5.開発モードで動かす"></a>
<br>
<br>

## 5.開発モードで動かす

- yarn dev を実行して、`http://localhost:3000/` を開くと `Welcome to Next.js!` の画面が表示される。

```
$ yarn dev
```

<a id="6.SSRを実装"></a>
<br>
<br>

## 6.SSR を実装

- page ディレクトリに ssr.tsx のファイルを追加する。
- サンプル実装で Yes か No で答えてくれる API を利用 https://yesno.wtf/
- SSR ソース：https://github.com/ka-yamao/next-firebase/blob/main/pages/ssr.tsx
- axios の追加もお願いします。`yarn add axios`

```

const URL = `https://yesno.wtf/api`;

type Data = {
  answer: string;
  forced: boolean;
  image: string;
};

type Props = {
  data: Data;
};

export const getServerSideProps: GetServerSideProps = async (context) => {
  const res = await axios.get<Data>(URL);
  const data = res.status === 200 ? res.data : undefined;
  return {
    props: {
      data,
    } as Props,
  };
};

export default function Ssr({ data }: Props) {
  const { answer, forced, image } = data;
  return (

      // 省略

      <main className={styles.main}>
        <h1>SSR</h1>
        <p>
          YesかNoで答えてくれるAPI １万回に１回、maybeが返ることもあるらしい。
        </p>
        <p>{answer}</p>
        <Image src={image} width={300} height={300} alt="logo" />
      </main>

      // 省略

  );
}
```

<a id="7.SSGを実装"></a>
<br>
<br>

## 7.SSG を実装

- SSG のソース：https://github.com/ka-yamao/next-firebase/blob/main/pages/%5Bid%5D.tsx
- `/ssg1`、`/ssg2` のダイナミックリーティングで実装、SSG なのでビルド時に生成され、リロードしても表示が変わらない。

```
const URL = `https://yesno.wtf/api`;

type Data = {
  answer: string;
  forced: boolean;
  image: string;
};

type PathParams = {
  id: string;
};

type PageProps = {
  id: string;
  data: Data;
};

export const getStaticPaths: GetStaticPaths<PathParams> = async () => {
  const paths = [{ params: { id: "ssg1" } }, { params: { id: "ssg2" } }];
  return {
    paths,
    fallback: false,
  };
};

export const getStaticProps: GetStaticProps<PageProps> = async (context) => {
  const { id } = context.params as PathParams;
  const res = await axios.get<Data>(URL);
  const props: PageProps = {
    id,
    data: res.data,
  };

  return { props };
};

// ページコンポーネントの実装
const Ids: React.FC<PageProps> = ({ id, data }: PageProps) => {
  const { answer, forced, image } = data;
  return (
    <div className={styles.container}>

     // 省略

      <main className={styles.main}>
        <h1>SSG {id}ページ</h1>
        <p>
          YesかNoで答えてくれるAPI １万回に１回、maybeが返ることもあるらしい。
        </p>
        <p>{answer}</p>
        <Image src={image} width={300} height={300} alt="logo" />
      </main>

      // 省略

  );
};
export default Ids;
```

<a id="8.Next.jsビルド"></a>
<br>
<br>

## 8.Next.js ビルド

- `/ssg1`と`/ssg2`がちゃんと生成されているかログで確認する。

```
# package.json の script の build に `next build` が設定されているので yarn で実行する。
$ yarn build
```

#### ログ

```
yarn build
yarn run v1.17.0
$ next build
info  - Linting and checking validity of types
info  - Creating an optimized production build
info  - Compiled successfully
info  - Collecting page data
info  - Generating static pages (5/5)
info  - Finalizing page optimization

Route (pages)                              Size     First Load JS
┌ ○ /                                      1.07 kB        77.6 kB
├   /_app                                  0 B            73.1 kB
├ ● /[id] (2524 ms)                        959 B          77.5 kB
├   ├ /ssg2 (1265 ms)
├   └ /ssg1 (1259 ms)
├ ○ /404                                   181 B          73.3 kB
├ λ /api/hello                             0 B            73.1 kB
└ λ /ssr                                   927 B          77.5 kB
+ First Load JS shared by all              73.4 kB
  ├ chunks/framework-8c5acb0054140387.js   45.4 kB
  ├ chunks/main-b482fffd82fa7e1c.js        26.7 kB
  ├ chunks/pages/_app-3893aca8cac41098.js  296 B
  ├ chunks/webpack-8fa1640cc84ba8fe.js     750 B
  └ css/ab44ce7add5c3d11.css               247 B

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)
```

<a id="9.Firebaseへデプロイする"></a>
<br>
<br>

## 9.Firebase へデプロイする

- ログの途中に`npm install --save firebase-functions@latest` という警告が出てるが、`Please note that there will be breaking changes when you upgrade.`アップグレードしたら動かなくなるかもとある。アップグレードしたらほんとに動かなくなります。

```
$ firebase deploy
```

#### ログ

```
Detected a Next.js codebase. This is an experimental integration, proceed with caution.

info  - Linting and checking validity of types
info  - Creating an optimized production build
info  - Compiled successfully
info  - Collecting page data
info  - Generating static pages (5/5)
info  - Finalizing page optimization

Route (pages)                              Size     First Load JS
┌ ○ /                                      1.07 kB        77.6 kB
├   /_app                                  0 B            73.1 kB
├ ● /[id] (2888 ms)                        959 B          77.5 kB
├   ├ /ssg2 (1451 ms)
├   └ /ssg1 (1437 ms)
├ ○ /404                                   181 B          73.3 kB
├ λ /api/hello                             0 B            73.1 kB
└ λ /ssr                                   927 B          77.5 kB
+ First Load JS shared by all              73.4 kB
  ├ chunks/framework-8c5acb0054140387.js   45.4 kB
  ├ chunks/main-b482fffd82fa7e1c.js        26.7 kB
  ├ chunks/pages/_app-3893aca8cac41098.js  296 B
  ├ chunks/webpack-8fa1640cc84ba8fe.js     750 B
  └ css/ab44ce7add5c3d11.css               247 B

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)


up to date in 2s

94 packages are looking for funding
  run `npm fund` for details

=== Deploying to 'プロジェクトID'...

i  deploying functions, hosting
i  functions: ensuring required API cloudfunctions.googleapis.com is enabled...
i  functions: ensuring required API cloudbuild.googleapis.com is enabled...
i  artifactregistry: ensuring required API artifactregistry.googleapis.com is enabled...
✔  functions: required API cloudbuild.googleapis.com is enabled
✔  artifactregistry: required API artifactregistry.googleapis.com is enabled
✔  functions: required API cloudfunctions.googleapis.com is enabled
i  functions: preparing codebase firebase-frameworks-プロジェクトID for deployment
⚠  functions: package.json indicates an outdated version of firebase-functions. Please upgrade using npm install --save firebase-functions@latest in your functions directory.
⚠  functions: Please note that there will be breaking changes when you upgrade.
i  functions: Loaded environment variables from .env.
i  functions: preparing .firebase/プロジェクトID/functions directory for uploading...
i  functions: packaged /Users/HogeHoge/workspace/next-firebase/.firebase/プロジェクトID/functions (44.16 MB) for uploading
i  functions: ensuring required API run.googleapis.com is enabled...
i  functions: ensuring required API eventarc.googleapis.com is enabled...
i  functions: ensuring required API pubsub.googleapis.com is enabled...
i  functions: ensuring required API storage.googleapis.com is enabled...
✔  functions: required API run.googleapis.com is enabled
✔  functions: required API eventarc.googleapis.com is enabled
✔  functions: required API pubsub.googleapis.com is enabled
✔  functions: required API storage.googleapis.com is enabled
i  functions: generating the service identity for pubsub.googleapis.com...
i  functions: generating the service identity for eventarc.googleapis.com...
✔  functions: .firebase/プロジェクトID/functions folder uploaded successfully
i  hosting[プロジェクトID]: beginning deploy...
i  hosting[プロジェクト]: found 23 files in .firebase/プロジェクトID/hosting
✔  hosting[プロジェクトID]: file upload complete
i  functions: updating Node.js 16 function firebase-frameworks-プロジェクトID:ssrプロジェクトID(us-central1)...
✔  functions[firebase-frameworks-プロジェクトID:ssrプロジェクトID(us-central1)] Successful update operation.
Function URL (firebase-frameworks-プロジェクトID:ssrプロジェクトID4a5f3(us-central1)): https://ssrプロジェクトID-fdsafsa233-uc.a.run.app
i  functions: cleaning up build files...
i  hosting[プロジェクトID]: finalizing version...
✔  hosting[プロジェクトID]: version finalized
i  hosting[プロジェクトID]: releasing new version...
✔  hosting[プロジェクトID]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/プロジェクトID/overview
Hosting URL: https://プロジェクトID.web.app

```

<a id="10.表示確認"></a>
<br>
<br>

## 10.表示確認

- Hosting の URL を開いて確認：`https://プロジェクト ID.web.app`
  - Next.js の画面が表示される。
- SSR の確認 `https://プロジェクト ID.web.app/ssr` を
  - API で取得した Yes or No の画像などが表示される。
  - リロードのたびに変わることを確認
- https://プロジェクト ID.web.app/ssg1 、

<a id="11.Firebaseを確認"></a>
<br>
<br>

## 11.Firebase を確認

- https://console.firebase.google.com/?hl=ja
  - Hosting にプロジェクトがデプロイされていること
  - Functions に ssr の関数がデプロイされていること

<a id="まとめ"></a>
<br>
<br>

## まとめ

Firebase のデプロイが楽ちんで便利になりました。
Vercel へデプロイ、自前のサーバーにデプロイするもの面倒で Firebase のほうが個人的に使い勝手がいいので 10 月 25 日の発表はとれもうれしかった。
Analytics もあるし、最近 Firebase を利用しないケースがないので Hosting を使っての Web アプリ開発もどんどんやってみたい。
SEO を気にしつつ Next.js で Web アプリを作っていきたい。

<a id="デプロイのときハマった"></a>
<br>
<br>

## デプロイのときハマった

- Firebase CLI ツールが古かった影響でデプロイのときこんなログがでました。11.16.1 のバージョンだとうまくいきました。

```

Unable to find a valid endpoint for function `ssrプロジェクトID`, but still including it in the config

```

<a id="参考資料"></a>
<br>
<br>

## 参考資料

- Next.js を統合する ： https://firebase.google.com/docs/hosting/nextjs

<a id="サンプルソース"></a>
<br>
<br>

## サンプルソース

- Github ： https://github.com/ka-yamao/next-firebase/
